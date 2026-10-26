<script>
    import { onMount } from 'svelte';
    import * as THREE from 'three';
    import italyData from '../italy.json';

    // Controls for the globe appearance (configure directly here or pass as props)
    let {
        globeColor = "#757575",
        wiggleSize = 0.2,        // Controls the amplitude / size of the squiggles (0 = smooth, 1.0 = standard handdrawn, >1 = larger wiggles)
        wiggleTurbulence = 1.0,  // Controls the frequency / turbulence of the squiggles (higher = denser, shakier squiggles)
        wiggleSpeed = 1.0        // Controls the speed of the hand-drawn line boil / shake animation (0 = static sketch)
    } = $props();

    let container = $state();
    let lineMaterial;

    // Reactively update shader uniforms when variables change
    $effect(() => {
        if (lineMaterial && lineMaterial.uniforms) {
            lineMaterial.uniforms.color.value.set(globeColor);
            lineMaterial.uniforms.wiggleSize.value = wiggleSize;
            lineMaterial.uniforms.wiggleTurbulence.value = wiggleTurbulence;
            lineMaterial.uniforms.wiggleSpeed.value = wiggleSpeed;
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

        // Globe group (upright, no design tilt)
        const globeGroup = new THREE.Group();
        scene.add(globeGroup);

        // Build Latitude and Longitude wireframe lines
        const radius = 2;
        const positions = [];
        // High segment count for smooth, detailed hand-drawn squiggles
        const segmentsPerCircle = 256;

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

        // ShaderMaterial with variable hand-drawn wiggle controls and front-center fade
        lineMaterial = new THREE.ShaderMaterial({
            uniforms: {
                color: { value: new THREE.Color(globeColor) },
                time: { value: 0 },
                baseOpacity: { value: 0.85 },
                radius: { value: radius },
                fadeRadius: { value: 1.6 },
                wiggleSize: { value: wiggleSize },
                wiggleTurbulence: { value: wiggleTurbulence },
                wiggleSpeed: { value: wiggleSpeed }
            },
            vertexShader: `
                uniform float time;
                uniform float wiggleSize;
                uniform float wiggleTurbulence;
                uniform float wiggleSpeed;

                varying vec3 vWorldPosition;
                varying vec3 vLocalPos;

                // Simplex noise implementation for GLSL
                vec3 mod289(vec3 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
                vec4 mod289(vec4 x) { return x - floor(x * (1.0 / 289.0)) * 289.0; }
                vec4 permute(vec4 x) { return mod289(((x * 34.0) + 1.0) * x); }
                vec4 taylorInvSqrt(vec4 r) { return 1.79284291400159 - 0.85373472095314 * r; }

                float snoise(vec3 v) {
                    const vec2 C = vec2(1.0 / 6.0, 1.0 / 3.0);
                    const vec4 D = vec4(0.0, 0.5, 1.0, 2.0);

                    vec3 i  = floor(v + dot(v, C.yyy));
                    vec3 x0 = v - i + dot(i, C.xxx);

                    vec3 g = step(x0.yzx, x0.xyz);
                    vec3 l = 1.0 - g;
                    vec3 i1 = min(g.xyz, l.zxy);
                    vec3 i2 = max(g.xyz, l.zxy);

                    vec3 x1 = x0 - i1 + C.xxx;
                    vec3 x2 = x0 - i2 + C.yyy;
                    vec3 x3 = x0 - D.yyy;

                    i = mod289(i);
                    vec4 p = permute(permute(permute(
                                i.z + vec4(0.0, i1.z, i2.z, 1.0))
                            + i.y + vec4(0.0, i1.y, i2.y, 1.0))
                            + i.x + vec4(0.0, i1.x, i2.x, 1.0));

                    float n_ = 0.142857142857;
                    vec3  ns = n_ * D.wyz - D.xzx;

                    vec4 j = p - 49.0 * floor(p * ns.z * ns.z);

                    vec4 x_ = floor(j * ns.z);
                    vec4 y_ = floor(j - 7.0 * x_);

                    vec4 x = x_ * ns.x + ns.yyyy;
                    vec4 y = y_ * ns.x + ns.yyyy;
                    vec4 h = 1.0 - abs(x) - abs(y);

                    vec4 b0 = vec4(x.xy, y.xy);
                    vec4 b1 = vec4(x.zw, y.zw);

                    vec4 s0 = floor(b0) * 2.0 + 1.0;
                    vec4 s1 = floor(b1) * 2.0 + 1.0;
                    vec4 sh = -step(h, vec4(0.0));

                    vec4 a0 = b0.xzyw + s0.xzyw * sh.xxyy;
                    vec4 a1 = b1.xzyw + s1.xzyw * sh.zzww;

                    vec3 p0 = vec3(a0.xy, h.x);
                    vec3 p1 = vec3(a0.zw, h.y);
                    vec3 p2 = vec3(a1.xy, h.z);
                    vec3 p3 = vec3(a1.zw, h.w);

                    vec4 norm = taylorInvSqrt(vec4(dot(p0, p0), dot(p1, p1), dot(p2, p2), dot(p3, p3)));
                    p0 *= norm.x;
                    p1 *= norm.y;
                    p2 *= norm.z;
                    p3 *= norm.w;

                    vec4 m = max(0.6 - vec4(dot(x0, x0), dot(x1, x1), dot(x2, x2), dot(x3, x3)), 0.0);
                    m = m * m;
                    return 42.0 * dot(m * m, vec4(dot(p0, x0), dot(p1, x1), dot(p2, x2), dot(p3, x3)));
                }

                void main() {
                    vLocalPos = position;

                    // Stop-motion style line boil for hand-drawn frame jitter (8 fps * wiggleSpeed)
                    float boilFrame = floor(time * 8.0 * wiggleSpeed) * 0.125;
                    vec3 boilSeed = vec3(boilFrame * 0.37, boilFrame * 0.61, boilFrame * 0.83);

                    // Turbulence scales the spatial frequency of the squiggles
                    float turb = max(0.01, wiggleTurbulence);

                    // Multi-frequency organic squiggles and wobbles
                    // 1. Low frequency unevenness
                    float wobbleLow = snoise(position * (2.2 * turb));

                    // 2. Medium frequency squiggly ripples
                    float wobbleMed = snoise((position + boilSeed * 0.15) * (8.5 * turb));

                    // 3. High frequency shaky hand tremor
                    float wobbleHigh = snoise((position + boilSeed * 0.4) * (24.0 * turb));

                    // Combined radial displacement scaled by wiggleSize
                    float radialDisp = (wobbleLow * 0.045 + wobbleMed * 0.032 + wobbleHigh * 0.016) * wiggleSize;

                    // Lateral/tangential displacement so lines waver side-to-side
                    vec3 norm = normalize(position);
                    vec3 up = abs(norm.y) < 0.99 ? vec3(0.0, 1.0, 0.0) : vec3(1.0, 0.0, 0.0);
                    vec3 tangent = normalize(cross(norm, up));

                    float lat1 = snoise((position.zxy + boilSeed * 0.2) * (7.0 * turb));
                    float lat2 = snoise((position.yzx + boilSeed * 0.35) * (20.0 * turb));
                    float lateralDisp = (lat1 * 0.03 + lat2 * 0.015) * wiggleSize;

                    vec3 displaced = position + norm * radialDisp + tangent * lateralDisp;

                    vec4 worldPos = modelMatrix * vec4(displaced, 1.0);
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
                varying vec3 vLocalPos;

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

                    // Subtle hand-drawn ink / pencil pressure variance along the stroke
                    float pressure = 0.88 + 0.12 * sin(vLocalPos.x * 30.0 + vLocalPos.y * 25.0 + vLocalPos.z * 20.0);

                    float alpha = baseOpacity * alphaFactor * pressure;

                    if (alpha <= 0.005) discard;

                    gl_FragColor = vec4(color, alpha);
                }
            `,
            transparent: true,
            depthWrite: false
        });

        const wireframeGlobe = new THREE.LineSegments(geometry, lineMaterial);
        globeGroup.add(wireframeGlobe);

        // Setup places billboards from italy.json
        const textureLoader = new THREE.TextureLoader();
        textureLoader.crossOrigin = 'anonymous';

        // Uniform random distribution on sphere surface
        const getRandomSpherePoint = (r) => {
            const u = Math.random();
            const v = Math.random();
            const theta = u * 2.0 * Math.PI;
            const phi = Math.acos(2.0 * v - 1.0);
            const sinPhi = Math.sin(phi);
            return new THREE.Vector3(
                r * sinPhi * Math.cos(theta),
                r * Math.cos(phi),
                r * sinPhi * Math.sin(theta)
            );
        };

        // Pick well-spaced random spots on the globe for each place
        const randomSpots = [];
        const minSeparation = 1.4; // minimum distance between spots on radius 2 sphere
        for (let i = 0; i < italyData.length; i++) {
            let candidate;
            let attempts = 0;
            while (attempts < 100) {
                candidate = getRandomSpherePoint(radius);
                const tooClose = randomSpots.some(p => p.distanceTo(candidate) < minSeparation);
                if (!tooClose || attempts > 80) break;
                attempts++;
            }
            randomSpots.push(candidate);
        }

        const billboards = [];

        italyData.forEach((place, index) => {
            // Invisible anchor point attached to the rotating globe
            const point = new THREE.Object3D();
            point.position.copy(randomSpots[index]);
            globeGroup.add(point);

            const item = {
                point,
                sprite: null,
                texture: null,
                material: null,
                aspect: 1.4 // sensible default until image loads
            };

            if (place.globeImage) {
                const texture = textureLoader.load(place.globeImage, (tex) => {
                    tex.colorSpace = THREE.SRGBColorSpace;
                    tex.generateMipmaps = true;
                    tex.minFilter = THREE.LinearMipmapLinearFilter;
                    const img = tex.image;
                    if (img && img.naturalWidth && img.naturalHeight) {
                        item.aspect = img.naturalWidth / img.naturalHeight;
                    } else if (img && img.width && img.height) {
                        item.aspect = img.width / img.height;
                    }
                });

                const material = new THREE.SpriteMaterial({
                    map: texture,
                    transparent: true,
                    depthWrite: false
                });

                const sprite = new THREE.Sprite(material);
                point.add(sprite);

                item.sprite = sprite;
                item.texture = texture;
                item.material = material;
            }

            billboards.push(item);
        });

        // Interaction state
        let isDragging = false;
        let prevMousePos = { x: 0, y: 0 };
        let velocityX = 0;
        let velocityY = 0;
        let lastMoveTime = 0;
        const idleSpeed = 0.002;
        const clock = new THREE.Clock();

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

            // Rotate globe directly (upright polar spin on Y, tilt on X)
            globeGroup.rotation.y += deltaX * 0.005;
            globeGroup.rotation.x += deltaY * 0.005;

            // Clamp vertical tilt
            globeGroup.rotation.x = Math.max(-Math.PI * 0.35, Math.min(Math.PI * 0.35, globeGroup.rotation.x));

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
        const tempWorldPos = new THREE.Vector3();
        const tempNormal = new THREE.Vector3();
        const tempToCam = new THREE.Vector3();

        const animate = () => {
            animationFrameId = requestAnimationFrame(animate);

            // Update shader time for hand-drawn line boil
            if (lineMaterial && lineMaterial.uniforms) {
                lineMaterial.uniforms.time.value = clock.getElapsedTime();
            }

            // Update billboard sizes according to their position on the globe
            const viewHeight = container.clientHeight || height || 600;
            const fovRad = THREE.MathUtils.degToRad(camera.fov);

            for (const item of billboards) {
                if (!item.sprite) continue;

                // World position of the invisible anchor point on the globe
                item.point.getWorldPosition(tempWorldPos);

                // Normal on the sphere surface (sphere centered at 0, 0, 0)
                tempNormal.copy(tempWorldPos).normalize();

                // Vector from the point on the globe towards the camera
                tempToCam.copy(camera.position).sub(tempWorldPos).normalize();

                // Dot product: 1.0 at front face (facing camera), -1.0 at back face
                const dot = tempNormal.dot(tempToCam);

                // Interpolate from 0.0 (backface) to 1.0 (front face)
                const t = THREE.MathUtils.clamp((dot + 1.0) * 0.5, 0.0, 1.0);

                // Vary size: 20px on backface, full (~200px) on front face
                const targetPxHeight = THREE.MathUtils.lerp(20, 200, t);

                // Vary opacity: 10% on backface, 100% on front face
                item.material.opacity = THREE.MathUtils.lerp(0, 1.0, t);

                // Convert target pixel height to world height for perspective camera
                const dist = camera.position.distanceTo(tempWorldPos);
                const frustumHeight = 2.0 * dist * Math.tan(fovRad * 0.5);
                const worldHeight = (targetPxHeight / viewHeight) * frustumHeight;
                const worldWidth = worldHeight * item.aspect;

                item.sprite.scale.set(worldWidth, worldHeight, 1.0);
            }

            if (!isDragging) {
                // Apply momentum
                globeGroup.rotation.y += velocityX;
                globeGroup.rotation.x += velocityY;

                // Damp momentum
                velocityX *= 0.95;
                velocityY *= 0.95;

                // Subtle idle spin around polar axis
                globeGroup.rotation.y += idleSpeed;

                // Keep tilt clamped
                globeGroup.rotation.x = Math.max(-Math.PI * 0.35, Math.min(Math.PI * 0.35, globeGroup.rotation.x));
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
            for (const item of billboards) {
                if (item.texture) item.texture.dispose();
                if (item.material) item.material.dispose();
            }
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
