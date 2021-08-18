<template>
  <main>
      <div class="machine">
          <div class="reel" v-for="reel in reels" :key="reel.id">
              <div class="column-of-symbols" :class="{spinning: isSpinning}">
                  <div v-for="symbol in reel.symbols" :key="symbol.id" class="symbol">
                      <img :src="require('@/assets/images/' + getSymbolSrc(symbol.value))" />
                  </div>
              </div>
          </div>
          <canvas class="lineVisualiser"></canvas>
      </div>
      <div>
          <span>Lines: {{ lines.length }} | Bet: {{ bet }} | Money: {{ money }} | Single win: {{ singleWin }}</span>
          <button v-if="isGameLoaded" @click="spin">Spin</button>
      </div>
      <div class="lazy-load">
          <img v-for="symbol in allSymbols"
          :key="symbol.id"
          class="loading-image"
          @load="loadImage($event, symbol)"
          :src="require('@/assets/images/7.png')">
      </div>
      <div v-if="errorLoading" class="error-message">Oops! Something went wrong.</div>
      <div v-if="singleWin >= bet * 10">Big win!</div>
  </main>
</template>

<script>
export default {
    name: "Game",
    data() {
        return {
            singleWin: 0,
            money: 1000000,
            bet: 30000,
            errorLoading: false,
            isGameLoaded: false,
            isSpinning: false,
            canStopSpinning: false,
            images: [],
            grid: {
                rows: 4,
                columns: 5
            },
            reelSpinInterval: null,
            reels: [
                {
                    id: 1,
                    symbols: [],
                    marginBottom: -536,
                    canStopSpinning: false
                },
                {
                    id: 2,
                    symbols: [],
                    marginBottom: -536,
                    canStopSpinning: false
                },
                {
                    id: 3,
                    symbols: [],
                    marginBottom: -536,
                    canStopSpinning: false
                },
                {
                    id: 4,
                    symbols: [],
                    marginBottom: -536,
                    canStopSpinning: false
                },
                {
                    id: 5,
                    symbols: [],
                    marginBottom: -536,
                    canStopSpinning: false
                }
            ],
            allSymbols: [
                {
                    id: 1,
                    image: '4.png',
                    values: {
                        3: 2,
                        4: 4,
                        5: 10
                    }
                },
                {
                    id: 2,
                    image: '5.png',
                    values: {
                        3: 2,
                        4: 4,
                        5: 10
                    }
                },
                {
                    id: 3,
                    image: '6.png',
                    values: {
                        3: 2,
                        4: 4,
                        5: 10
                    }
                },
                {
                    id: 4,
                    image: '7.png',
                    values: {
                        3: 5,
                        4: 10,
                        5: 40
                    }
                },
                {
                    id: 5,
                    image: 'kota1.png',
                    values: {
                        3: 8,
                        4: 20,
                        5: 75
                    }
                },
                {
                    id: 6,
                    image: 'kota2.png',
                    values: {
                        3: 8,
                        4: 20,
                        5: 75
                    }
                },
                {
                    id: 7,
                    image: 'kota3.png',
                    values: {
                        3: 5,
                        4: 10,
                        5: 40
                    }
                },
                {
                    id: 8,
                    image: 'kota4.png',
                    values: {
                        3: 5,
                        4: 10,
                        5: 40
                    }
                },
                {
                    id: 9,
                    image: 'kota-wild.png',
                    isWild: true,
                    values: {
                        3: 10,
                        4: 30,
                        5: 200
                    }
                }
            ],
            lines: [
                [0, 0, 0, 0, 0],
                [3, 3, 3, 3, 3],
                [1, 1, 1, 1, 1],
                [2, 2, 2, 2, 2],
                [0, 1, 2, 1, 0],
                [1, 2, 3, 2, 1],
                [2, 1, 0, 1, 2],
                [3, 2, 1, 2, 3],
                [0, 1, 0, 1, 0],
                [1, 2, 1, 2, 1],
                [2, 3, 2, 3, 2],
                [1, 0, 1, 0, 1],
                [2, 1, 2, 1, 2],
                [3, 2, 3, 2, 3],
                [0, 1, 1, 1, 0],
                [1, 2, 2, 2, 1],
                [2, 3, 3, 3, 2],
                [1, 0, 0, 0, 1],
                [2, 1, 1, 1, 2],
                [3, 2, 2, 2, 3],
                [0, 0, 1, 0, 0],
                [1, 1, 2, 1, 1],
                [2, 2, 3, 2, 2],
                [1, 1, 0, 1, 1],
                [2, 2, 1, 2, 2],
                [3, 3, 2, 3, 3],
                [0, 0, 1, 2, 3],
                [3, 3, 2, 1, 0],
                [3, 2, 1, 0, 1],
                [0, 1, 2, 3, 2],
                [0, 0, 2, 0, 0],
                [1, 1, 3, 1, 1],
                [2, 2, 0, 2, 2],
                [3, 3, 1, 3, 3],
                [1, 3, 3, 3, 1],
                [2, 0, 0, 0, 2],
                [1, 0, 2, 0, 1],
                [2, 1, 3, 1, 2],
                [1, 2, 0, 2, 1],
                [2, 3, 1, 3, 2],
                [0, 2, 0, 2, 0],
                [1, 3, 1, 3, 1],
                [2, 0, 2, 0, 2],
                [3, 1, 3, 1, 3],
                [0, 2, 1, 2, 0],
                [1, 3, 2, 3, 1],
                [2, 0, 1, 0, 2],
                [3, 1, 2, 1, 3],
                [0, 3, 2, 3, 0],
                [3, 0, 1, 0, 3]
            ]
        }
    },
    methods: {
        clearLines() {
            let canvas = document.querySelector('.lineVisualiser');
            let ctx = canvas.getContext('2d')
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        },
        drawLines(paths) {
            let canvas = document.querySelector('.lineVisualiser');
            let ctx = canvas.getContext('2d');

            for(let i = 0; i < paths.length; i++) {
                let path = paths[i];
                ctx.strokeStyle = this.randomColor();
                ctx.moveTo(0, path[0] * 134 + 67);
                ctx.lineTo(198 + 198 / 2, path[1] * 134 + 67);
                ctx.lineTo((198 * 2) + 198 / 2, path[2] * 134 + 67);
                ctx.lineTo((198 * 3) + 198 / 2, path[3] * 134 + 67);
                ctx.lineTo((198 * 4) + 198 / 2, path[4] * 134 + 67);
                ctx.lineTo((198 * 5) + 198 / 2, path[5] * 134 + 67);
                ctx.stroke();
            }
        },
        randomColor() {
            let r = Math.floor(Math.random() * 255) + 1,
                g = Math.floor(Math.random() * 255) + 1,
                b = Math.floor(Math.random() * 255) + 1;
            return `rgb(${r}, ${g}, ${b})`;
        },
        loadImage(event, symbol) {
            let image = {
                id: symbol.id,
                element: event.currentTarget,
            }
            this.images.push(image);
        },
        randomNumber(min, max) {
            return Math.floor(Math.random() * (max - min)) + min + 1;
        },
        removeSymbolFromReel(reel) {
            reel.symbols.splice(reel.symbols.length-1, 1);
            reel.marginBottom += 134;
        },
        addSymbolToReels(id, reel) {
            let rn = this.randomNumber(1, this.allSymbols.length);
                let obj = {
                    id: id,
                    value: rn
                }
            reel.symbols = [obj, ...reel.symbols];
        },
        getNumOfSymbols() {
            return this.reels[this.reels.length-1].symbols.length;
        },
        setReelSpinInterval() {
            let self = this;
            this.reelSpinInterval = setInterval(function() {
                self.reels.forEach(function(reel) {
                    let reelElement = document.querySelectorAll('.reel')[reel.id - 1].children[0];
                    if(reel.canStopSpinning === false) {
                        if(reelElement) {
                            reel.marginBottom -= 64;
                            reelElement.style.marginBottom = `${reel.marginBottom}px`;
                        }
                        let id = 0;
                        if(self.getNumOfSymbols() < 24) {
                            do {
                                id++;
                            }
                            while(self.getSymbolById(reel.id, id))
                            self.addSymbolToReels(id, reel);
                        } else if(reel.marginBottom < -256) {
                            self.removeSymbolFromReel(reel);
                        }
                    } else {
                        reelElement.classList.add('spin-end');
                        setTimeout(function() {
                            reelElement.classList.remove('spin-end');
                        }, 500);
                        reel.marginBottom = -530;
                        reelElement.style.marginBottom = `${reel.marginBottom}px`;
                    }
                });
            }, 1000/60);
        },
        getSymbolById(reelId, symbolId) {
            let reel = this.reels.filter(function(key) {
                return key.id == reelId;
            })[0];
            if(reel) {
                let symbol = reel.symbols.filter(function(key) {
                    return key.id == symbolId;
                })[0];
                return symbol;
            }
            return undefined;
        },
        checkLines() {
            let visibleSymbols = [
                [], [], [], []
            ];
            this.reels.forEach(function(reel) {
                let i = 0;
                reel.symbols.forEach(function(symbol, index) {
                    if(index == 19 || index == 18 || index == 17 || index == 16) {
                        visibleSymbols[i].push(symbol.value);
                        i++;
                    }
                });
            });

            let matches = [];
            let currentMatch = null;
            let count = 0;
            let paths = [];

            this.lines.forEach(function(line, lineIndex) {
                count = 0;
                currentMatch = null;
                for(let i = 0; i < line.length; i++) {
                    let value = line[i];
                    if(i == 0) currentMatch = visibleSymbols[value][i];

                    if(visibleSymbols[value][i] != currentMatch || visibleSymbols[value].isWild === true) {
                        break;
                    }
                    count++;
                }
                if(count >= 3) {
                    paths.push(line);
                    matches.push({id: lineIndex, matches: count, symbol: currentMatch});
                }
            });

            this.drawLines(paths);
            this.addMoney(matches);

            console.log(visibleSymbols);
        },
        addMoney(matches) {
            let self = this;
            matches.forEach(function(match) {
                let howMuch = self.allSymbols[match.symbol - 1].values[match.matches];
                self.singleWin += Math.floor((howMuch * self.bet) / self.lines.length);
            });
            this.money += self.singleWin;
        },
        stopSpinning() {
            this.canStopSpinning = true;
            this.reels.forEach(function(reel, index) {
                setTimeout(function() { reel.canStopSpinning = true; }, 300 * (index + 1))
            });
            let self = this;

            setTimeout(function() {
                self.isSpinning = false;
                clearInterval(self.reelSpinInterval);
                self.checkLines();
            }, 2000);
        },
        spin() {
            if(this.isSpinning === false) {
                this.clearLines();
                this.singleWin = 0;
                this.money -= this.bet;
                this.reels.forEach(function(reel) {
                    reel.canStopSpinning = false;
                });
                this.isSpinning = true;
                this.canStopSpinning = false;
                this.setReelSpinInterval();
                this.stopSpinning();
            }
        },
        prepareMachine() {
            let self = this;
            this.reels.forEach(function(reel) {
                for(let i = 0; i < self.grid.rows * 6; i++) {
                    self.addSymbolToReels(i, reel);
                }
            });
        },
        getSymbolSrc(symbolId) {
            return this.allSymbols.filter(function(key) { return key.id == symbolId })[0].image;
        }
    },
    watch: {
        images: function() {
            if(this.images.length == this.allSymbols.length) {
                this.prepareMachine();
                this.isGameLoaded = true;
            }
        }
    },
    mounted() {
        let canvas = document.querySelector('.lineVisualiser');
        canvas.width = 996;
        canvas.height = 554;
        // document.querySelector('.spinning').addEventListener('transitionend', () => {
        //     console.log('Transition ended');
        // });
    }
}
</script>

<style>

.lazy-load {
    display: none;
}

.machine {
    background: #000;
    display: inline-flex;
    justify-content: space-evenly;
    padding: 6px;
    width: 996px;
    height: 554px;
    position: relative;
}

.machine .lineVisualiser {
    position: absolute;
    background: transparent;
    user-select: none;
    pointer-events: none;
    width: 100%;
    height: 100%;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
}

.reel {
    width: 192px;
    height: calc(128px * 4 + 4 * 6);
    background: #fff;
    position: relative;
    overflow: hidden;
    background: #000;
}

.reel:not(:first-child) {
    margin-left: 6px;
}

.reel .column-of-symbols {
    position: absolute;
    left: 0;
    bottom: 0;
    margin-bottom: 6px;
    min-height: 3216px;
}

.reel .column-of-symbols.spin-end {
    transition: margin-bottom .5s;
    transition-timing-function: cubic-bezier(0.11, 0.99, 0.32, 1.275);
}

/* .reel .column-of-symbols.spinning {
    animation: spinning 2s;
    animation-fill-mode: forwards;  
    animation-delay: 0;
} */

.reel:nth-child(2) .column-of-symbols.spinning {
    animation-delay: .02s;
}

.reel:nth-child(3) .column-of-symbols.spinning {
    animation-delay: .04s;
}

.reel .column-of-symbols .symbol {
    width: 192px;
    height: 128px;
    margin-top: 6px;
}

@keyframes spinning {
    from {
        margin-bottom: 6px;
    }
    to {
        margin-bottom: calc(12 * -134px + 6px);
    }
}
</style>