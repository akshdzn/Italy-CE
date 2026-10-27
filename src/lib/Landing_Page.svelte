<script>
    import Globe from './Globe.svelte';

    let isDisabled = false;
</script>

<div class="container">
    <link rel="stylesheet" href="Landing_Page.css">
    <video autoplay muted loop playsinline id="bg-video">
      <source src="/Landing_Page.mp4" type="video/mp4">
      Your browser does not support HTML5 video.
    </video>
    <div class="Title"><h1>The Italian Atelier</h1></div>
</div>

<div class="globe">
    <Globe />
</div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Italianno&display=swap');
    .container {
        &.disable {
            display: none;
        }
        z-index: 100;
    }

    .globe {
        &.disable {
            display: block;
        }
        display: none;
    }

    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    #bg-video {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      object-fit: cover;
      z-index: -100;
    }
    .content {
      position: relative;
      z-index: 1;
      color: white;
      text-align: center;
      margin-top: 40vh;
    }
    .Title{
        position: absolute;
        top: 75%;
        left: 65%;
        transform: translate(-50%, -50%);
        padding: 5px 12px;
        border: 1px solid black;
        border-radius: 40px;
        background: rgba(255, 255, 255, 0.08);
         backdrop-filter: blur(3px);
        -webkit-backdrop-filter: blur(3px);
        white-space: nowrap;
    h1{
        font-family: 'Italianno', cursive;
        font-size: 96px;
        font-weight: normal;
        color: white;
        margin: 0;
        line-height: 1.1;
        position:center;
        margin-top:10px;
    }
    }
</style>
