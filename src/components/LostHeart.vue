<!--*************************************************************************************************************

    TEMPLATE/HTML AREA

*************************************************************************************************************-->
<template>
    <div>
        <div :style="game.score.style">{{ game.score.totalPoints }}</div>
        <div :style="game.score.timer.style" v-if="game.score.timer.value != 0">
                {{ String(game.score.timer.value / 1000).substring(0, 3) }}
        </div>
        
        <div :style="heart.points.style" v-if="heart.points.value != 0">+{{ heart.points.value }}</div>
        <img src="../assets/img/heart.gif" :style="heart.style"
             @dragstart="heartDragStart" @drag="heartDrag" @dragend="heartDragEnd"
             @mousedown="heartMouseDown" @mouseup="heartMouseUp">
        
        <img src="../assets/img/hole.gif" :style="hole.style" v-for="hole of holes.list" :key="hole.id">
    </div>
</template>
<!--*************************************************************************************************************

     JAVASCRIPT/OBJECTS AREA

*************************************************************************************************************-->
<script>
export default {
    name: 'LostHeart',
    data: function() {
        return {
            game: {
                windowWidth  : window.innerWidth,
                windowHeight : window.innerHeight,
                loopTimeOut  : 3000,
                nextLevel    : 20,
                isFirstPlay  : true,
                audio: {
                    soundtrack: new Audio(require('../assets/audio/back.mp3'))
                },
                score: {
                    totalPoints   : 0,
                    pointsOK      : 1,
                    pointsPerfect : 3,
                    style: {
                        position      : 'absolute',
                        left          : '40px',
                        top           : '10px',
                        color         : 'red',
                        'font-size'   : '50px',
                        'font-weight' : 'bolder',
                        'font-family' : 'Verdana, Geneva, Tahoma, sans-serif'
                    },
                    timer: {
                        value: 0,
                        style: {
                            position      : 'absolute',
                            left          : '45px',
                            top           : '80px',
                            color         : 'red',
                            'font-size'   : '20px',
                            'font-weight' : 'normal',
                            'font-family' : 'Verdana, Geneva, Tahoma, sans-serif'
                        }
                    }
                }
            },
            heart: {
                style: {
                    position   : 'absolute',
                    opacity    : 1,
                    left       : '100px',
                    top        : '100px',
                    width      : '66px',
                    height     : '100px',
                    cursor     : 'grab',
                    'z-index'  : 100,
                    //border    : 'solid 1px yellow',               
                },
                audio: {
                    suck: new Audio(require('../assets/audio/suck.wav'))
                },
                points: {
                        value: 0,
                        style: {
                            position           : 'absolute',
                            left               : '100px',
                            top                : '100px',
                            width              : '66px',                            
                            'border-radius'    : '20px',
                            'background-color' : 'rgba(255, 0, 0, 0.7)',
                            'z-index'          : 101,
                            color              : 'white',
                            'font-size'        : '40px',
                            'font-weight'      : 'bolder',
                            'font-family'      : 'Verdana, Geneva, Tahoma, sans-serif'
                        }
                    },
                drag: {
                    mouseOffsetX : -1,
                    mouseOffsetY : -1,
                }
            },
            holes: {
                list             : [],
                maxHoles         : 1,
                deleteTimerMin   : 1000,
                deleteTimer      : 1000,
                defaultWidth     : 150,
                defaultHeight    : 150                
            }
        }
    },
    created: function() {
        setInterval(() => {
            this.createHoles()
            this.deleteHoles()
        }, this.game.loopTimeOut)
    },
    methods: {
        /******************************************************************************************************
         * createHoles(): 
         *      CALLS THE generateHole() METHOD holes.maxHoles TIMES
         *      ACTIVATES THE DECREASING TIMER
         * 
         ******************************************************************************************************/
        createHoles: function() {
            for(let i = 0; i < this.holes.maxHoles; i++) {
                this.generateHole()
            }
            let id = setInterval(() => {
                this.game.score.timer.value -= 100
                if(this.game.score.timer.value <= 0) {
                    this.game.score.timer.value = 0
                    clearInterval(id)
                }
            }, 100)
        },
        /******************************************************************************************************
         * generateHole(): 
         *      GENERATES A NEW HOLE AND ADDS IT TO THE LIST OF HOLES (holes.list)
         * 
         ******************************************************************************************************/
        generateHole: function() {
            let coords  = this.generateCoords()
            let newHole = {
                style: {
                    position : 'absolute',
                    left     : coords.x,
                    top      : coords.y,
                    width    : this.holes.defaultWidth  + 'px',
                    height   : this.holes.defaultHeight + 'px',
                    //border   : 'solid 1px yellow'
                }
            }
            this.holes.list.push(newHole)
        },
        /******************************************************************************************************
         * generateCoords(): 
         *      GENERATES AN OBJECT { x, y } WITH NEW RANDOM COORDS
         *      USED FOR HOLES AND HEART
         * 
         ******************************************************************************************************/
        generateCoords: function() {
            let maxX = this.game.windowWidth  - this.holes.defaultWidth
            let maxY = this.game.windowHeight - this.holes.defaultHeight
            let x    = Math.floor(Math.random() * maxX) + 'px'
            let y    = Math.floor(Math.random() * maxY) + 'px'
            return { x, y }
        },
        /******************************************************************************************************
         * deleteHoles(): 
         *      DELETES EVERY HOLE DISPLAYED
         *      IF totalPoints IS GREATER THAN nextLevel VALUE, ADD 1 HOLE TO THE NEXT ROUNDS OF HOLES
         *      THE nextLevel VALUE IS INCREASED
         * 
         ******************************************************************************************************/
        deleteHoles: function() {
            this.game.score.timer.value = Math.floor(Math.random() * (this.game.loopTimeOut - this.holes.deleteTimerMin)) + this.holes.deleteTimerMin
            this.holes.deleteTimer      = this.game.score.timer.value
            setTimeout(() => {
                this.holes.list = []
                if(this.game.score.totalPoints > this.game.nextLevel) {
                    this.holes.maxHoles++
                    this.game.nextLevel += this.game.nextLevel
                }
            }, this.holes.deleteTimer)
        },
        /******************************************************************************************************
         * 
         * 
         * 
         *             MOUSE HANDLING METHODS
         * 
         * 
         * 
         ******************************************************************************************************/
        heartMouseDown: function(mouseEvent) {
            this.heart.style.opacity     = 0.5
            this.heart.style.cursor      = 'grabbing'
            if(this.game.isFirstPlay) {
                this.game.audio.soundtrack.volume = 0.5
                this.game.audio.soundtrack.play()
                this.game.isFirstPlay = false
            }
        },
        heartDragStart: function(dragEvent) {
            this.heart.style.opacity     = 0.5
            this.heart.drag.mouseOffsetX = dragEvent.offsetX
            this.heart.drag.mouseOffsetY = dragEvent.offsetY
        },
        heartDrag: function(dragEvent) {
            this.heart.style.left   = (dragEvent.clientX - this.heart.drag.mouseOffsetX) + 'px'
            this.heart.style.top    = (dragEvent.clientY - this.heart.drag.mouseOffsetY) + 'px'
        },
        heartDragEnd: function(dragEvent) {
            this.heart.style.opacity = 1
            this.heart.style.cursor  = 'grab'
            this.heart.style.left    = (dragEvent.clientX - this.heart.drag.mouseOffsetX) + 'px'
            this.heart.style.top     = (dragEvent.clientY - this.heart.drag.mouseOffsetY) + 'px'
            this.checkCollisions()
        },
        heartMouseUp: function(mouseEvent) {
            this.heart.style.opacity = 1
            this.heart.style.cursor  = 'grab'
        },
        /******************************************************************************************************
         * checkCollisions(): 
         *      CALLS THE getHitBox(entity) METHOD WITH THE HEART AND THEN WITH EVERY HOLE DISPLAYED
         *      CALLS THE getPointsForCollisions(heartHB, holeHB) METHOD AND VERIFIES IF ANY points ARE RETURNED
         *      IF points IS NOT ZERO, CALLS THE doScoreAnimations(holeIndex, points) METHOD
         ******************************************************************************************************/
        checkCollisions: function() {
            let heartHB = this.getHitBox(this.heart)
            for(let i = 0; i < this.holes.list.length; i++) {
                let holeHB = this.getHitBox(this.holes.list[i])
                let points = this.getPointsForCollisions(heartHB, holeHB)
                if(points != 0) {
                    this.doScoreAnimations(i, points)
                    break
                }
            }
        },
        /******************************************************************************************************
         * doScoreAnimations(holeIndex, points): 
         *      CHANGES THE SCORE AFTER A SUCCESSFUL DROP OF THE HEART OVER A HOLE
         *      CALLS THE doHeartAnimation(holeIndex, points) METHOD
         ******************************************************************************************************/
        doScoreAnimations: function(holeIndex, points) {
            this.game.score.totalPoints += points
            this.doHeartAnimation(holeIndex, points)
        },
        /******************************************************************************************************
         * doHeartAnimation(holeIndex, points): 
         *      ANYMATES THE HEART AFTER A SUCCESSFUL DROP OVER A HOLE
         ******************************************************************************************************/
        doHeartAnimation: function(holeIndex, points) {
            let shinkIterations          = 50
            let newHeartWidth            = parseInt(this.heart.style.width)
            let newHeartHeight           = parseInt(this.heart.style.height)
            let shrinkWidthFactor        = parseFloat(newHeartWidth  / shinkIterations)
            let shrinkHeightFactor       = parseFloat(newHeartHeight / shinkIterations)
            let targetHole               = this.holes.list[holeIndex]
            this.heart.style.left        = parseInt(targetHole.style.left) + (parseInt(targetHole.style.width ) / 2) + 'px'
            this.heart.style.top         = parseInt(targetHole.style.top ) + (parseInt(targetHole.style.height) / 2) + 'px'
            this.heart.points.style.left = (parseInt(targetHole.style.left) + 40) + 'px'
            this.heart.points.style.top  = (parseInt(targetHole.style.top ) + 20) + 'px'
            this.heart.points.value      = points
            this.heart.audio.suck.play()
            let id = setInterval(() => {
                newHeartWidth  -= shrinkWidthFactor
                newHeartHeight -= shrinkHeightFactor
                this.heart.points.style.top = (parseFloat(this.heart.points.style.top) - 0.5) + 'px'
                this.heart.style.width  = parseInt(newHeartWidth)  + 'px'
                this.heart.style.height = parseInt(newHeartHeight) + 'px'
                if(newHeartWidth <= 0) {
                    this.doHoleDeleteAnimation(holeIndex)
                    this.resetHeart()
                    clearInterval(id)
                }
            }, 1)
        },
        /******************************************************************************************************
         * doHoleDeleteAnimation(holeIndex): 
         *      ANYMATES THE HOLE WITH INDEX holeIndex AFTER A SUCCESSFUL HEART DROP
         *      DELETES THE HOLE OF THE holes.list ARRAY
         ******************************************************************************************************/
        doHoleDeleteAnimation: function(holeIndex) {
            if(this.holes.list.length == 0)
                return
            let shinkIterations    = 20
            let newHoleWidth       = parseInt(this.holes.list[holeIndex].style.width)
            let newHoleHeight      = parseInt(this.holes.list[holeIndex].style.height)
            let shrinkWidthFactor  = parseFloat(newHoleWidth  / shinkIterations)
            let shrinkHeightFactor = parseFloat(newHoleHeight / shinkIterations)
            let id = setInterval(() => {
                if(this.holes.list.length == 0) {
                    clearInterval(id)
                    return
                }
                newHoleWidth  -= shrinkWidthFactor
                newHoleHeight -= shrinkHeightFactor
                this.holes.list[holeIndex].style.width  = parseInt(newHoleWidth)  + 'px'
                this.holes.list[holeIndex].style.height = parseInt(newHoleHeight) + 'px'
                if(newHoleWidth <= 0) {
                    this.holes.list.splice(holeIndex, 1)
                    clearInterval(id)
                }
            }, 1)
        },
        /******************************************************************************************************
         * resetHeart(): 
         *      RESETS THE HEART'S SIZE AFTER A SUCCESSFUL ANIMATION
         *      ON TOP OF THAT, RELOCATES THE HEART RANDOMLY
         ******************************************************************************************************/
        resetHeart: function() {
            let coords              = this.generateCoords()
            this.heart.points.value = 0
            this.heart.style.left   = coords.x
            this.heart.style.top    = coords.y
            this.heart.style.width  = '66px'
            this.heart.style.height = '100px'
        },
        /******************************************************************************************************
         * getPointsForCollisions(heartHB, holeHB): 
         *  THIS ALGORITHM VERIFIES IF THE heartHB (Heart Hit Box) IS COLLIDING WITH holeHB (Hole Hit Box)                              
         * - RETURNS pointsOK IF THE HEART WAS DROPPED ON ANY EDGE OF A HOLE
         * - RETURNS pointsPerfect IF THE HEART WAS DROPPED ON THE CENTER OF A HOLE
         * - RETURNS 0 IF THE HEART WAS DROPPED OUTSIDE OF A HOLE
         * 
         *      A HIT BOX LOOKS LIKE THIS:
         *      x1, y1 ------------- x2, y1
         *        |                     |
         *        |    Heart or Hole    |
         *        |       image         |
         *        |                     |
         *      x1, y2 ------------- x2, y2
         *  
         ******************************************************************************************************/
        getPointsForCollisions(heartHB, holeHB) {
            if(holeHB.x1 < heartHB.x1 && holeHB.x2 > heartHB.x2) {
                if(holeHB.y1 < heartHB.y1 && holeHB.y2 > heartHB.y2) {
                    return this.game.score.pointsPerfect
                }
            }
            if(holeHB.x1 >= heartHB.x1 && holeHB.x1 <= heartHB.x2) {
                if(holeHB.y1 >= heartHB.y1 && holeHB.y1 <= heartHB.y2) {
                    return this.game.score.pointsOK
                }
                if(holeHB.y1 <= heartHB.y1 && holeHB.y2 >= heartHB.y2) {
                    return this.game.score.pointsOK
                }
                if(holeHB.y2 >= heartHB.y1 && holeHB.y2 <= heartHB.y2) {
                    return this.game.score.pointsOK
                }
            }
            if(holeHB.x1 <= heartHB.x1 && holeHB.x2 >= heartHB.x2) {
                if(holeHB.y1 >= heartHB.y1 && holeHB.y1 <= heartHB.y2) {
                    return this.game.score.pointsOK
                }
                if(holeHB.y2 >= heartHB.y1 && holeHB.y2 <= heartHB.y2) {
                    return this.game.score.pointsOK
                }
            }
            if(holeHB.x2 >= heartHB.x1 && holeHB.x2 <= heartHB.x2) {
                if(holeHB.y1 >= heartHB.y1 && holeHB.y1 <= heartHB.y2) {
                    return this.game.score.pointsOK
                }
                if(holeHB.y1 <= heartHB.y1 && holeHB.y2 >= heartHB.y2) {
                    return this.game.score.pointsOK
                }
                if(holeHB.y2 >= heartHB.y1 && holeHB.y2 <= heartHB.y2) {
                    return this.game.score.pointsOK
                }
            }
            return 0
        },
        /***************************************************************************
         * getHitBox(entity): CALCULATES AND RETURNS AN OBJECT { x1, x2, y1, y2 } 
         *      WITH THE COORDS OF THE entity's HIT BOX 
         *      BASED ON THE POSITION AND SIZE OF THE entity (Heart or Hole)
         *      
         *      A HIT BOX LOOKS LIKE THIS:
         *      x1, y1 ------------- x2, y1
         *        |                     |
         *        |    Heart or Hole    |
         *        |       image         |
         *        |                     |
         *      x1, y2 ------------- x2, y2
         * 
         ****************************************************************************/
        getHitBox: function(entity) {
            let x1 = parseInt(entity.style.left)
            let x2 = x1 + parseInt(entity.style.width)
            let y1 = parseInt(entity.style.top)
            let y2 = y1 + parseInt(entity.style.height)
            return { x1, x2, y1, y2 }
        }
    }
}
</script>
<!--*************************************************************************************************************

     CSS/STYLES AREA

*************************************************************************************************************-->
<style scoped>
</style>