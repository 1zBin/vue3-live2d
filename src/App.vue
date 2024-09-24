<script setup>
import { onBeforeUnmount, onMounted, ref } from "vue";

const live2d = PIXI.live2d;

const canvas = ref(null);

let app;
let model;

onMounted(async () => {
  app = new PIXI.Application({
    view: canvas.value,
    // height: 500,
    // width: 500,
    autoDensity: true,
    antialias: true,
    resolution: window.devicePixelRatio,
    autoStart: true,
    resizeTo: window,
    transparent: true,
  });
  model = await live2d.Live2DModel.from("assets/shizuku/shizuku.model.json");

  app.stage.addChild(model);

  const scaleX = (innerWidth * 0.2) / model.width;
  const scaleY = (innerHeight * 0.4) / model.height;

  // fit the window
  model.scale.set(Math.min(scaleX, scaleY));

  model.x = innerWidth * 0.82;
  model.y = innerHeight * 0.6;

  draggable(model);
  addFrame(model);
  // addHitAreaFrames(model);

  model.on("hit", (hitAreas) => {
    console.log("hitAreas", hitAreas);

    if (hitAreas.includes("body")) {
      model.motion("tap_body");
    }

    if (hitAreas.includes("head")) {
      model.expression();
    }
  });
});

function draggable(model) {
  model.buttonMode = true;
  model.on("pointerdown", (e) => {
    model.dragging = true;
    model._pointerX = e.data.global.x - model.x;
    model._pointerY = e.data.global.y - model.y;
  });
  model.on("pointermove", (e) => {
    if (model.dragging) {
      model.position.x = e.data.global.x - model._pointerX;
      model.position.y = e.data.global.y - model._pointerY;
    }
  });
  model.on("pointerupoutside", () => (model.dragging = false));
  model.on("pointerup", () => (model.dragging = false));
}

function addFrame(model) {
  const foreground = PIXI.Sprite.from(PIXI.Texture.WHITE);
  foreground.width = model.internalModel.width;
  foreground.height = model.internalModel.height;
  foreground.alpha = 0.2;

  model.addChild(foreground);
}

function addHitAreaFrames(model) {
  const hitAreaFrames = new live2d.HitAreaFrames();

  model.addChild(hitAreaFrames);
}

onBeforeUnmount(() => {
  model?.destroy();
  app?.destroy();
});
</script>

<template>
  <div>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<style scoped></style>
