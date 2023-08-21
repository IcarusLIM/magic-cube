<template>
    <div class="container">
        <canvas id="cube-canvas"></canvas>
    </div>
</template>

<script setup lang="ts">
import {
    Animation,
    Engine,
    Scene,
    ArcRotateCamera,
    Vector3,
    HemisphericLight,
    AnimationGroup,
    QuinticEase
} from "babylonjs";
import { onMounted } from "vue";

function createSence() {
    const canvasElem = document.getElementById("cube-canvas") as HTMLCanvasElement
    const engine = new Engine(canvasElem, true, { stencil: true })
    const scene = new Scene(engine)
    // @ts-ignore
    const camera = new ArcRotateCamera("Camera", Math.PI / 4, Math.PI / 3, 8, BABYLON.Vector3.Zero(), scene)
    initLight(scene)

    // const box = BABYLON.MeshBuilder.CreateBox("box1", { width: 1, height: 1, depth: 1 })
    // console.log(box.position)

    // const box2 = BABYLON.MeshBuilder.CreateBox("box1", { width: 1, height: 1, depth: 1 })
    // box2.position.x = 3

    // let animation = new Animation(
    //     "rotateAnimation",
    //     "rotation.x",
    //     10,
    //     Animation.ANIMATIONTYPE_FLOAT,
    //     Animation.ANIMATIONLOOPMODE_CONSTANT
    // );
    // let keys = [];
    // keys.push({
    //     frame: 0,
    //     value: 0.39269908169872414
    // });
    // keys.push({
    //     frame: 30,
    //     value: 1.5707963267948966
    // });
    // animation.setKeys(keys);
    // animation.setEasingFunction(new QuinticEase())

    // let animationGroup = new AnimationGroup("group")
    // animationGroup.addTargetedAnimation(animation, box)
    // animationGroup.addTargetedAnimation(animation, box2)

    // animationGroup.play()

    engine.runRenderLoop(() => {
        scene.render();
    });
}


function initLight(scene: Scene) {
    let light1 = new HemisphericLight(
        "light",
        new Vector3(-1, 0, 0),
        scene
    );
    let light2 = new HemisphericLight(
        "light",
        new Vector3(1, 0, 0),
        scene
    );
    let light3 = new HemisphericLight(
        "light",
        new Vector3(0, 1, 0),
        scene
    );
    let light4 = new HemisphericLight(
        "light",
        new Vector3(0, -1, 0),
        scene
    );
}


onMounted(() => {
    if (!Engine.isSupported()) {
        window.alert("It seems you must try this page in a newer browser :)");
    } else {
        createSence()
    }
})

</script>

<style scoped>
.container {
    width: 100%;
    height: 80%;
}

.container canvas {
    width: 100%;
    height: 80%;
}
</style>