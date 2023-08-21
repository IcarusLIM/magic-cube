<template>
    <div class="container">
        <canvas id="cube-canvas"></canvas>
        <!-- <div class="control-panel">
            <div>
                <button mat-mini-fab (click)="start()" color="primary" :disabled="isFreeze">
                    <mat-icon aria-label="Play">play_arrow</mat-icon>
                </button>
                <n-button *ngIf="cube != null" mat-icon-button (click)="autoSolve()" color="accent"
                    :disabled="isFreeze || cube.isSolved()">
                    <mat-icon aria-label="Auto Solve">vpn_key</mat-icon>
                </n-button>
            </div>
            <div>
                <button mat-icon-button @click="resetCamera()">
                    <mat-icon aria-label="Reset">refresh</mat-icon>
                </button>
                <button mat-icon-button (click)="rotateCamera(-1)">
                    <mat-icon aria-label="Rotate left">keyboard_arrow_left</mat-icon>
                </button>
                <button mat-icon-button (click)="rotateCamera(1)">
                    <mat-icon aria-label="Rotate right">keyboard_arrow_right</mat-icon>
                </button>
                <button mat-icon-button (click)="switchCamera()">
                    <n-icon size="40" color="#0e7a0d">
                        <game-controller />
                    </n-icon>
                    <mat-icon *ngIf="cameraPosition[1] === 1" aria-label="Top look">keyboard_arrow_up</mat-icon>
                    <mat-icon *ngIf="cameraPosition[1] === 0" aria-label="Bottom look">keyboard_arrow_down</mat-icon>
                </button>
            </div>
        </div> -->
    </div>
</template>

<script setup lang="ts">
import {
    Animation,
    Engine,
    Scene,
    ArcRotateCamera,
    Mesh,
    Vector3,
    HemisphericLight,
    MeshBuilder,
    Plane,
    StandardMaterial,
    Space,
    AnimationGroup,
    Color3,
    ActionManager,
    Light
} from "babylonjs";
import { GameControllerOutline, GameController } from '@vicons/ionicons5'
import Cube from "../lib/Cube";
import { easingFunction } from "../lib/helpers";
import { AxisEnum } from "../lib/enums";
import {
    getPoints,
    getPoissonsEquation,
    getTranslateVector,
    directionToRotatePieces
} from "../lib/helpers";
import { onMounted, onUnmounted, ref } from "vue";

const ORDER_NUMBER = 3;
const DIRECTION_PLANE_WIDTH = 5;

const isFreeze = ref(false)

let canvasElem: HTMLCanvasElement,
    engine: Engine,
    scene: Scene,
    lights: Light[],
    camera: ArcRotateCamera,
    cube: Cube;

function initEngine() {
    canvasElem = document.getElementById("cube-canvas") as HTMLCanvasElement
    engine = new Engine(canvasElem, true, { stencil: true })
    scene = new Scene(engine)
    scene.clearColor = new BABYLON.Color4(1, 1, 1, 0)
    // @ts-ignore
    scene.actionManager = new BABYLON.ActionManager(scene)
    camera = new ArcRotateCamera(
        "Camera",
        cameraAlphas[0],
        cameraBetas[0],
        8,
        // @ts-ignore
        BABYLON.Vector3.Zero(),
        scene
    )
    camera.inputs.clear();
    initLight()
    cube = createCube(ORDER_NUMBER)
}

let directionPlane: Mesh | null;
let preRotateParams: [AxisEnum, number, number] | null;
let pickInfo: { pickStartTime: number; } | null;
let cameraAnimationGroup: AnimationGroup;
let cameraPosition: [number, number] = [0, 0];
const alphaOffset = Math.PI / 4;
const cameraAlphas = [
    alphaOffset,
    Math.PI / 2 + alphaOffset,
    Math.PI + alphaOffset,
    (Math.PI * 3) / 2 + alphaOffset
];
const cameraBetas = [Math.PI / 3, (Math.PI * 2) / 3];

function translateCameraPosition([alphaIndex, betaIndex]: [number, number]): [
    number,
    number
] {
    return [cameraAlphas[alphaIndex], cameraBetas[betaIndex]];
}

function rotateCameraAnimationGroup(
    [alpha, beta]: [number, number],
    speed: number,
    camera: ArcRotateCamera,
    isClockwise: boolean
) {
    let animationAlpha = new Animation(
        "rotateCameraAlpha",
        "alpha",
        60,
        Animation.ANIMATIONTYPE_FLOAT,
        Animation.ANIMATIONLOOPMODE_CONSTANT
    );
    let oldAlpha = camera.alpha;
    if (isClockwise && oldAlpha > alpha) {
        oldAlpha -= Math.PI * 2;
    } else if (!isClockwise && oldAlpha < alpha) {
        oldAlpha += Math.PI * 2;
    }
    animationAlpha.setKeys([
        {
            frame: 0,
            value: oldAlpha
        },
        {
            frame: speed,
            value: alpha
        }
    ]);
    let animationBeta = new Animation(
        "rotateCameraBeta",
        "beta",
        60,
        Animation.ANIMATIONTYPE_FLOAT,
        Animation.ANIMATIONLOOPMODE_CONSTANT
    );
    let oldBeta = camera.beta;
    animationBeta.setKeys([
        {
            frame: 0,
            value: oldBeta
        },
        {
            frame: speed,
            value: beta
        }
    ]);
    animationAlpha.setEasingFunction(easingFunction);
    animationBeta.setEasingFunction(easingFunction);
    let animationGroup = new AnimationGroup("rotateCamera");
    animationGroup.addTargetedAnimation(animationAlpha, camera);
    animationGroup.addTargetedAnimation(animationBeta, camera);
    return animationGroup;
}

function initLight() {
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

function startRendering() {
    engine.runRenderLoop(() => {
        scene.render();
    });
}

function rotateCamera(value: number) {
    if (cameraAnimationGroup) {
        cameraAnimationGroup.stop();
    }
    cameraPosition[0] += value;
    if (cameraPosition[0] > 3) {
        cameraPosition[0] %= 4;
    } else if (cameraPosition[0] < 0) {
        cameraPosition[0] = (cameraPosition[0] % 3) + 4;
    }
    cameraAnimationGroup = rotateCameraAnimationGroup(
        translateCameraPosition(cameraPosition),
        30,
        camera,
        value > 0
    ).play();
}

function resetCamera() {
    if (cameraAnimationGroup) {
        cameraAnimationGroup.stop();
    }
    let animationGroup = rotateCameraAnimationGroup(
        translateCameraPosition([0, 0]),
        30,
        camera,
        true
    ).play();
    cameraAnimationGroup = animationGroup;
    cameraPosition = [0, 0];
}

function switchCamera() {
    if (cameraAnimationGroup) {
        cameraAnimationGroup.stop();
    }
    cameraPosition[1] = cameraPosition[1] === 0 ? 1 : 0;
    cameraAnimationGroup = rotateCameraAnimationGroup(
        translateCameraPosition(cameraPosition),
        30,
        camera,
        true
    ).play();
}

function debounce(func: any, time: number | null) {
    time = time || 100;
    let timer: number;
    return function (event: Event) {
        if (timer) clearTimeout(timer);
        timer = setTimeout(func, time!, event);
    };
}

function onResize(event?: Event) {
    // console.log(window.innerHeight, window.innerWidth)
    const size = Math.min(window.innerHeight, window.innerWidth)
    canvasElem.style.width = size + "px"
    canvasElem.style.height = size + "px"
    engine.resize();
}

function pickStartCB(event: MouseEvent) {
    if (isFreeze.value) return;
    pickInfo = {
        pickStartTime: new Date().getTime()
    };
    if (directionPlane) {
        directionPlane.dispose();
        directionPlane = null;
    }
    let pickResult = scene.pick(event.clientX, event.clientY);
    if (!pickResult.hit) return;
    // console.log("pickStart", pickResult.pickedMesh, pickResult.pickedMesh!.getIndices())
    let indices = pickResult.pickedMesh!.getIndices();
    let index0 = indices![pickResult.faceId * 3];
    let index1 = indices![pickResult.faceId * 3 + 1];
    let index2 = indices![pickResult.faceId * 3 + 2];
    // console.log("pickStart", pickResult.faceId, index0, index1, index2)
    let [p1, p2, p3] = getPoints(pickResult.pickedMesh!, [
        index0,
        index1,
        index2
    ]);
    // (x, y, z, distance) point to touch plane
    let [a, b, c, d] = getPoissonsEquation(p1, p2, p3);
    // console.log("pickStart", "p1~p3", p1, p2, p3, "a~c", a, b, c, d)
    let sourcePlane = new Plane(a, b, c, 0);
    let plane = MeshBuilder.CreatePlane(
        "directionPlane",
        {
            height: DIRECTION_PLANE_WIDTH,
            width: DIRECTION_PLANE_WIDTH,
            sourcePlane: sourcePlane,
            sideOrientation: Mesh.DOUBLESIDE
        },
        scene
    );
    plane.isPickable = false;
    plane.isVisible = true;
    let mat = new StandardMaterial("mat", scene);
    mat.emissiveColor = Color3.White();
    mat.wireframe = true;
    plane.material = mat;
    let tv = getTranslateVector(p1, p2, p3);
    plane.translate(tv, 1, Space.WORLD);
    setTimeout(function () {
        plane.isPickable = true;
    }, 10);
    directionPlane = plane;
};

function pickingCB(event: MouseEvent) {
    if (isFreeze.value) return;
    let pickResult = scene.pick(event.clientX, event.clientY, mesh => {
        if (mesh == directionPlane && mesh.isPickable === true) {
            return true;
        }
        return false;
    });
    if (pickResult.hit) {
        let direction = pickResult.pickedPoint!.add(
            pickResult.pickedMesh!.getAbsolutePosition().negate()
        );
        let normal = pickResult.getNormal(true);
        // console.log(direction, normal)
        let rotateParams = directionToRotatePieces(
            direction,
            // @ts-ignore
            normal,
            pickResult.pickedMesh!.getAbsolutePosition(),
            ORDER_NUMBER
        );
        if (direction.length() > 1) {
            cube.preRotate(rotateParams[0], rotateParams[1], rotateParams[2]);
            preRotateParams = rotateParams;
        } else {
            preRotateParams = null;
            cube.clearPreRotate();
        }
    }
};

function pickStopCB(event: MouseEvent) {
    console.log("pickstop", preRotateParams)
    if (isFreeze.value) return;
    if (directionPlane) {
        directionPlane.dispose();
        directionPlane = null;
    }
    console.log("pickstop", preRotateParams)
    if (preRotateParams) {
        if (
            pickInfo != null &&
            new Date().getTime() - pickInfo.pickStartTime > 100
        ) {
            console.log("preRotateParams", preRotateParams)
            cube.rotatePieces(
                preRotateParams[0],
                preRotateParams[1],
                preRotateParams[2] >= 0
            );
        }
    }
    preRotateParams = null;
    cube.clearPreRotate();
};

function initController() {
    canvasElem.addEventListener("pointerdown", pickStartCB);
    canvasElem.addEventListener("pointerup", pickStopCB);
    canvasElem.addEventListener("pointermove", pickingCB);
}

// start() {
//     if (!this.cube.isSolved()) {
//         let dialogRef = this.dialog.open(CubeInfoDialog, {
//             width: "350px",
//             data: {
//                 title: "CUBE_INF_DLG.RESTART",
//                 content: "CUBE_INF_DLG.RESTART_CAUTION",
//                 no: "CUBE_INF_DLG.CONTINUE",
//                 yes: "CUBE_INF_DLG.RESTART"
//             }
//         });
//         dialogRef.afterClosed().subscribe((isRestart: boolean) => {
//             if (isRestart) {
//                 this.restart();
//             }
//         });
//     } else {
//         this.restart();
//     }
// }

// restart() {
//     if (!this.cube.isSolved()) {
//         this.cube.reset();
//     }
//     this.cube.scramble();
// }

// autoSolve() {
//     let dialogRef = this.dialog.open(CubeInfoDialog, {
//         width: "350px",
//         data: {
//             title: "CUBE_INF_DLG.ANSWER",
//             content: this.cube.getAnswer(),
//             no: "CUBE_INF_DLG.CLOSE",
//             yes: "CUBE_INF_DLG.AUTO_RESTORE"
//         }
//     });
//     dialogRef.afterClosed().subscribe((isRestart: boolean) => {
//         if (isRestart) {
//             this.cube.rotateByLetters(this.cube.getAnswer());
//         }
//     });
// }


function createCube(orderNum: number) {
    return new Cube(scene, orderNum, {
        rotatingStopCB: () => {
            isFreeze.value = false;
            // if (cube.isSolved()) {
            //     let dialogRef = this.dialog.open(CubeInfoDialog, {
            //         width: "350px",
            //         data: {
            //             title: "CUBE_INF_DLG.CHEER_UNLOCK",
            //             content: "CUBE_INF_DLG.NEW_START",
            //             no: "CUBE_INF_DLG.CLOSE",
            //             yes: "CUBE_INF_DLG.RESTART"
            //         }
            //     });
            //     dialogRef.afterClosed().subscribe((isRestart: boolean) => {
            //         if (isRestart) {
            //             this.restart();
            //         }
            //     });
            // }
        },
        rotatingStartCB: () => {
            isFreeze.value = true;
        }
    });
}

onMounted(() => {
    if (!Engine.isSupported()) {
        window.alert("It seems you must try this page in a newer browser :)");
    } else {
        initEngine()
        startRendering()
        initController()
        onResize()
    }
    window.addEventListener('resize', onResize);
})

onUnmounted(() => {
    window.removeEventListener('resize', onResize);
})

</script>

<style scoped>
.container {
    width: 100%;
    height: 80%;
}
</style>