<template>
<div>
   <section class="row">
        <div class="small-6 columns">
            <h1 class="text-center">YOU</h1>
            <div class="healthbar">
                <div
                  class="healthbar text-center"
                  style="background-color: green; margin: 0; color: white;"
                  :style="{width: playerHealth + '%'}">
                  {{ playerHealth }}
                </div>
            </div>
        </div>
        <div class="small-6 columns">
            <h1 class="text-center">MONSTER</h1>
            <div class="healthbar">
                <div
                  class="healthbar text-center"
                  style="background-color: green; margin: 0; color: white;"
                  :style="{width: monsterHealth + '%'}">
                  {{ monsterHealth }}
                </div>
            </div>
        </div>
    </section>
    <section class="row controls" v-if="!gameIsRunning">
        <div class="small-12 columns">
            <button id="start-game" @click="startGame">START NEW GAME</button>
        </div>
    </section>
    <section class="row controls" v-else>
        <div class="small-12 columns">
            <button id="attack" @click="attack">ATTACK</button>
            <button id="special-attack" @click="specialAttack">SPECIAL ATTACK</button>
            <button id="heal" @click="heal">HEAL</button>
            <button id="give-up" @click="giveUp">GIVE UP</button>
        </div>
    </section>
    <section class="row log" v-if="turns.length > 0 ">
        <div class="small-12 columns">
            <ul>
                <li v-for="(turn,i) in turns" v-bind:key="i"
                    :class="{'mega-turn': turn.damage > 10, 'player-turn': turn.isPlayer, 'monster-turn': !turn.isPlayer}">
                  {{ turn.text }}
                </li>
            </ul>
        </div>
    </section>
</div>
</template>

<script>
export default {
  name: 'MainView',
  data () {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      gameIsRunning: false,
      turns: []
    }
  },
  methods: {
    startGame: function () {
      this.gameIsRunning = true
      this.playerHealth = 100
      this.monsterHealth = 100
      this.turns = []
      this.generateAttack()
    },
    generateAttack: async function () {
      if (!this.gameIsRunning) return
      await this.sleep(2000)
      var value = this.calculateDamage(0, 100)
      if (value >= 87) {
        this.specialAttack()
      } else {
        this.attack()
      }
      this.generateAttack()  
    },
    giveUp: function () {
      this.gameIsRunning = false
    },
    attack: function () {
      this.playerAttacks(3, 10)
    },
    specialAttack: function () {
      this.playerAttacks(11, 20)
    },
    sleep: function(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    heal: function () {
      var heal = 0
      if (this.playerHealth <= 90) {
        heal = 10
        this.playerHealth += heal
      } else {
        heal = 100 - this.playerHealth
        this.playerHealth = heal
      }
      this.addLog(true, 'heals', heal)
      this.monsterAttacks()
    },
    playerAttacks: function (min, max) {
      var damage = this.calculateDamage(min, max)
      this.monsterHealth -= damage
      var action = ''
      if (damage > 10) {
        action = 'hits critical'
      } else {
        action = 'hits'
      }
      this.addLog(true, action, damage)
      if (this.checkWin()) {
        return
      }
      this.monsterAttacks()
    },
    monsterAttacks: function () {
      var damage = this.calculateDamage(5, 12)
      this.playerHealth -= damage
      this.addLog(false, 'hits', damage)
      this.checkWin()
    },
    addLog: function (isPlayer, action, value) {
      var text = ''
      if (isPlayer) {
        text = 'Player ' + action + ' Monster for ' + value + 'HP'
      } else {
        text = 'Monster ' + action + ' Player for ' + value + 'HP'
      }
      this.turns.unshift({
        isPlayer: isPlayer,
        text: text,
        damage: value
      })
    },
    checkWin: function () {
      if (this.monsterHealth <= 0) {
        if (confirm('You won')) {
          this.startGame()
        } else {
          this.gameIsRunning = false
        }
        return true
      } else if (this.playerHealth <= 0) {
        if (confirm('You lost')) {
          this.startGame()
        } else {
          this.gameIsRunning = false
        }
        return true
      }
      return false
    },
    calculateDamage: function (min, max) {
      return Math.max(Math.floor(Math.random() * max) + 1, min)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.text-center {
    text-align: center;
}

.healthbar {
    width: 80%;
    height: 40px;
    background-color: #eee;
    margin: auto;
    transition: width 500ms;
}

.controls, .log {
    margin-top: 30px;
    text-align: center;
    padding: 10px;
    border: 1px solid #ccc;
    box-shadow: 0px 3px 6px #ccc;
}

.turn {
    margin-top: 20px;
    margin-bottom: 20px;
    font-weight: bold;
    font-size: 22px;
}

.log ul {
    list-style: none;
    font-weight: bold;
    text-transform: uppercase;
}

.log ul li {
    margin: 5px;
}

.log ul .player-turn {
    color: blue;
    background-color: #e4e8ff;
}

.log ul .monster-turn {
    color: red;
    background-color: #ffc0c1;
}

.log ul .mega-turn {
    color: orange;
    background-color: lightgoldenrodyellow;
}

button {
    font-size: 20px;
    background-color: #eee;
    padding: 12px;
    box-shadow: 0 1px 1px black;
    margin: 10px;
}

#start-game {
    background-color: #aaffb0;
}

#start-game:hover {
    background-color: #76ff7e;
}

#attack {
    background-color: #ff7367;
}

#attack:hover {
    background-color: #ff3f43;
}

#special-attack {
    background-color: #ffaf4f;
}

#special-attack:hover {
    background-color: #ff9a2b;
}

#heal {
    background-color: #aaffb0;
}

#heal:hover {
    background-color: #76ff7e;
}

#give-up {
    background-color: #ffffff;
}

#give-up:hover {
    background-color: #c7c7c7;
}
</style>
