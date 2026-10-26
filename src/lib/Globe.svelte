<script>
    import { onMount } from 'svelte';
    import * as THREE from 'three';

    let { globeColor = "#000000" } = $props();

    let container = $state();
    let lineMaterial;

    $effect(() => {
        if (lineMaterial && lineMaterial.uniforms) {
            lineMaterial.uniforms.color.value.set(globeColor);
        }
    });

    onMount(() => {
        if (!container) return;

        // Scene setup
        const scene = new THREE.Scene();

        const width = container.clientWidth || 600;
        const height = container.clientHeight || 600;

        const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 1000);
        camera.position.z = 6;

        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.setSize(width, height);
        container.appendChild(renderer.domElement);

        const tiltGroup = new THREE.Group();
        tiltGroup.rotation.z = (Math.PI / 180);
        tiltGroup.rotation.x = 0.2; // slight forward tilt
        scene.add(tiltGroup);

        // Globe group that spins around its polar axis
        const globeGroup = new THREE.Group();
        tiltGroup.add(globeGroup);

        // Build Latitude and Longitude wireframe lines
        const radius = 2;
        const positions = [];
        const segmentsPerCircle = 128;

        // Latitude lines (parallels) every 15 degrees from -75° to 75°
        const latStep = 15;
        for (let lat = -75; lat <= 75; lat += latStep) {
            const phi = (lat * Math.PI) / 180;
            const r = radius * Math.cos(phi);
            const y = radius * Math.sin(phi);

            for (let i = 0; i < segmentsPerCircle; i++) {
                const theta1 = (i / segmentsPerCircle) * Math.PI * 2;
                const theta2 = ((i + 1) / segmentsPerCircle) * Math.PI * 2;

                positions.push(
                    r * Math.cos(theta1), y, r * Math.sin(theta1),
                    r * Math.cos(theta2), y, r * Math.sin(theta2)
                );
            }
        }

        // Longitude lines (meridians) every 15 degrees (12 full great circles = 24 meridians)
        const lonStep = 15;
        const numMeridians = 180 / lonStep;
        for (let m = 0; m < numMeridians; m++) {
            const lambda = (m * lonStep * Math.PI) / 180;

            for (let i = 0; i < segmentsPerCircle; i++) {
                const alpha1 = (i / segmentsPerCircle) * Math.PI * 2;
                const alpha2 = ((i + 1) / segmentsPerCircle) * Math.PI * 2;

                positions.push(
                    radius * Math.sin(alpha1) * Math.cos(lambda),
                    radius * Math.cos(alpha1),
                    radius * Math.sin(alpha1) * Math.sin(lambda),

                    radius * Math.sin(alpha2) * Math.cos(lambda),
                    radius * Math.cos(alpha2),
                    radius * Math.sin(alpha2) * Math.sin(lambda)
                );
            }
        }

        const geometry = new THREE.BufferGeometry();
        geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));

        // ShaderMaterial that fades out lines as they come to the front center
        lineMaterial = new THREE.ShaderMaterial({
            uniforms: {
                color: { value: new THREE.Color(globeColor) },
                baseOpacity: { value: 0.85 },
                radius: { value: radius },
                fadeRadius: { value: 5 }
            },
            vertexShader: `
                varying vec3 vWorldPosition;

                void main() {
                    vec4 worldPos = modelMatrix * vec4(position, 1.0);
                    vWorldPosition = worldPos.xyz;
                    gl_Position = projectionMatrix * viewMatrix * worldPos;
                }
            `,
            fragmentShader: `
                uniform vec3 color;
                uniform float baseOpacity;
                uniform float radius;
                uniform float fadeRadius;
                varying vec3 vWorldPosition;

                void main() {
                    // Front-center point on the sphere facing the camera (0, 0, radius)
                    vec3 frontCenter = vec3(0.0, 0.0, radius);
                    float distToFront = distance(vWorldPosition, frontCenter);

                    // Fade out as the line approaches the front center
                    float frontFade = smoothstep(0.15, fadeRadius, distToFront);

                    // 2D distance from center axis for clean fade through the center
                    float distXY = length(vWorldPosition.xy);
                    float backFade = smoothstep(0.15, 1.3, distXY);

                    // Distinguish front and back hemisphere
                    float isFront = step(0.0, vWorldPosition.z);
                    float alphaFactor = isFront > 0.5 ? frontFade : (0.35 * backFade);

                    float alpha = baseOpacity * alphaFactor;

                    if (alpha <= 0.005) discard;

                    gl_FragColor = vec4(color, alpha);
                }
            `,
            transparent: true,
            depthWrite: false
        });

        const wireframeGlobe = new THREE.LineSegments(geometry, lineMaterial);
        globeGroup.add(wireframeGlobe);

        // Interaction state
        let isDragging = false;
        let prevMousePos = { x: 0, y: 0 };
        let velocityX = 0;
        let velocityY = 0;
        let lastMoveTime = 0;
        const idleSpeed = 0.002;

        const onPointerDown = (e) => {
            isDragging = true;
            prevMousePos = { x: e.clientX, y: e.clientY };
            velocityX = 0;
            velocityY = 0;
            lastMoveTime = Date.now();
            try {
                container.setPointerCapture(e.pointerId);
            } catch (_) {}
        };

        const onPointerMove = (e) => {
            if (!isDragging) return;

            const deltaX = e.clientX - prevMousePos.x;
            const deltaY = e.clientY - prevMousePos.y;

            // Rotate globe
            globeGroup.rotation.y += deltaX * 0.005;
            tiltGroup.rotation.x += deltaY * 0.005;

            // Clamp vertical tilt
            tiltGroup.rotation.x = Math.max(-Math.PI * 0.35, Math.min(Math.PI * 0.35, tiltGroup.rotation.x));

            // Record velocity
            velocityX = deltaX * 0.005;
            velocityY = deltaY * 0.005;

            prevMousePos = { x: e.clientX, y: e.clientY };
            lastMoveTime = Date.now();
        };

        const onPointerUp = (e) => {
            if (!isDragging) return;
            isDragging = false;

            // If stationary for more than 100ms before release, kill flick velocity
            if (Date.now() - lastMoveTime > 100) {
                velocityX = 0;
                velocityY = 0;
            } else {
                // Clamp velocity to prevent extreme spinning
                velocityX = Math.max(-0.08, Math.min(0.08, velocityX));
                velocityY = Math.max(-0.05, Math.min(0.05, velocityY));
            }

            try {
                container.releasePointerCapture(e.pointerId);
            } catch (_) {}
        };

        container.addEventListener('pointerdown', onPointerDown);
        window.addEventListener('pointermove', onPointerMove);
        window.addEventListener('pointerup', onPointerUp);
        window.addEventListener('pointercancel', onPointerUp);

        // Handle resizing
        const resizeObserver = new ResizeObserver((entries) => {
            for (const entry of entries) {
                const { width: w, height: h } = entry.contentRect;
                if (w > 0 && h > 0) {
                    camera.aspect = w / h;
                    camera.updateProjectionMatrix();
                    renderer.setSize(w, h, false);
                }
            }
        });
        resizeObserver.observe(container);

        // Animation loop
        let animationFrameId;
        const animate = () => {
            animationFrameId = requestAnimationFrame(animate);

            if (!isDragging) {
                // Apply momentum
                globeGroup.rotation.y += velocityX;
                tiltGroup.rotation.x += velocityY;

                // Damp momentum
                velocityX *= 0.95;
                velocityY *= 0.95;

                // Subtle idle spin
                globeGroup.rotation.y += idleSpeed;

                // Keep tilt clamped
                tiltGroup.rotation.x = Math.max(-Math.PI * 0.35, Math.min(Math.PI * 0.35, tiltGroup.rotation.x));
            }

            renderer.render(scene, camera);
        };
        animate();

        // Cleanup
        return () => {
            cancelAnimationFrame(animationFrameId);
            resizeObserver.disconnect();

            container.removeEventListener('pointerdown', onPointerDown);
            window.removeEventListener('pointermove', onPointerMove);
            window.removeEventListener('pointerup', onPointerUp);
            window.removeEventListener('pointercancel', onPointerUp);

            geometry.dispose();
            lineMaterial.dispose();
            renderer.dispose();
            if (renderer.domElement && renderer.domElement.parentNode) {
                renderer.domElement.parentNode.removeChild(renderer.domElement);
            }
        };
    });
</script>

<div class="globe-container" bind:this={container}></div>

<style>
    .globe-container {
        width: 100%;
        height: 100%;
        min-height: 400px;
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: grab;
        touch-action: none;
        user-select: none;
    }

    .globe-container:active {
        cursor: grabbing;
    }

    :global(.globe-container canvas) {
        display: block;
        width: 100% !important;
        height: 100% !important;
    }
</style>
