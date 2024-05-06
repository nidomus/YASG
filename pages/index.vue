<template>
    <section class="hero is-fullheight">
        <div class="hero-head">
            <nav class="navbar has-shadow">
                <div class="navbar-brand">
                    <div class="navbar-item">

                        <h1 class="title has-text-light ">
                            <span>
                                SUD<span id="logo-icon">
                                    <!-- <Icon name="iconamoon:number-9-circle-fill" size="3rem" /> -->
                                    <img src="../assets/images/wave.jpg" alt="">
                                </span>KU!
                            </span>
                        </h1>

                    </div>
                </div>
            </nav>
        </div>
        <div class="hero-body">

            <div class="columns is-multiline is-centered">

                <div class="column is-4-desktop is-12-tablet-only is-12-touch ">

                    <div id="sudoku" v-if="!loading">
                        <div v-for="cell, index in sudoku"
                            class="cell is-clickable is-size-4-desktop is-6-mobile has-text-centered"
                            :data-number="cell.number" :data-column="cell.column" :data-row="cell.row"
                            @click="selectedCell = cell"
                            :class="{ 'editable': cell.editable, 'selected': selectedCell === cell, 'marcada': ehMarcada(cell), 'wrong': cell.conflicts > 0 }">


                            <div id="annotations" v-show="cell.number === ''">
                                <div class="annotation" v-for="n in 9" :class="{ 'visible': cell.annotations[n - 1] }">
                                    {{ n }}
                                </div>
                            </div>

                        </div>
                    </div>
                    <div class="" v-else>
                        <div class="button is-static is-loading is-large is-fullwidth" id="loader">
                        </div>
                    </div>

                </div>

                <div class="column is-4-desktop  is-12-touch">

                    <div class="columns m-0 is-multiline is-centered is-mobile is-vcentered " id="actions">
                        <div class="column is-12 is-hidden-touch pb-0">

                            <div class="field has-addons" id="generate">
                                <div class="control is-expanded">
                                    <button class="button is-fullwidth has-text-white is-success" @click="getSudoku()">
                                        Novo Sudoku
                                    </button>
                                </div>
                                <div class="control">
                                    <div class="select">
                                        <select name="difficulty" id="" v-model="difficultySelect">
                                            <option value="easy">Fácil</option>
                                            <option value="medium">Médio</option>
                                            <option value="hard">Difícil</option>
                                        </select>
                                    </div>
                                </div>
                            </div>
                            <hr class="mb-0">
                        </div>

                        <div class="column is-12 py-5">
                            <p class="is-size-5-desktop is-size-6-touch icon-text" :class="{ 'visible': !loading }"
                                id="difficulty-text">
                                <span class="icon">
                                    <Icon name="streamline:brain-cognitive-solid" size="1rem" />
                                </span>
                                <span>
                                    Dificuldade:
                                    {{ difficulty === 'easy' ? 'Fácil' : difficulty === 'medium' ? 'Médio' : 'Difícil'
                                    }}
                                </span>
                            </p>


                        </div>

                        <div v-for="i in 9" class="column is-4-desktop px-1 is-2-touch">
                            <button class="button is-medium is-responsive  is-fullwidth"
                                @click="annotationMode ? addAnnotation($event, i - 1) : addValue($event, i)">
                                {{ i }}
                            </button>
                        </div>
                        <div class="column is-4-desktop is-2-touch  px-1">
                            <button class="button is-medium is-responsive 
                             is-fullwidth" @click="addValue($event, '')" id="erase-button">
                                <span class="is-hidden-desktop ">
                                    <Icon name="mdi:eraser" class="icon" />
                                </span>
                                <Icon name="mdi:eraser" size="1rem" class="icon is-hidden-touch is-size-5 " />
                                <span class="is-hidden-touch">

                                </span>
                            </button>
                        </div>
                        <div class="column is-4-desktop is-2-touch px-1">
                            <button class="button is-medium is-responsive is-light is-fullwidth"
                                :class="{ 'is-active': annotationMode }" @click="annotationMode = !annotationMode"
                                id="annotation-button">

                                <span class="is-hidden-desktop">
                                    <Icon name="mdi:pencil-outline" class="icon" />
                                </span>
                                <Icon name="mdi:pencil-outline" class="icon is-hidden-touch is-size-5" />
                                <span class="is-hidden-touch">

                                </span>

                            </button>
                        </div>

                        <div class="column is-4-desktop is-2-touch  px-1">
                            <button class="button is-medium is-responsive is-fullwidth" @click="reset()"
                                id="reset-button">

                                <span class="is-hidden-desktop">
                                    <Icon name="mdi:restart" class="icon" />
                                </span>
                                <Icon name="mdi:restart" class="icon is-hidden-touch is-size-5" />
                                <span class="is-hidden-touch">

                                </span>

                            </button>
                        </div>
                        <div class="column is-12 is-hidden-desktop px-0">
                            <hr>
                            <div class="field has-addons" id="generate">
                                <div class="control is-expanded">
                                    <button class="button is-fullwidth has-text-white is-success" @click="getSudoku()">
                                        Novo Sudoku
                                    </button>
                                </div>
                                <div class="control">
                                    <div class="select">
                                        <select name="difficulty" v-model="difficultySelect">
                                            <option value="easy">Fácil</option>
                                            <option value="medium">Médio</option>
                                            <option value="hard">Difícil</option>
                                        </select>
                                    </div>
                                </div>
                            </div>

                        </div>
                    </div>



                </div>
            </div>

        </div>

        <div class="hero-foot">
            <footer class="footer pb-5">
                <div class="content has-text-centered">
                    <p> Made by <strong>Nidomus</strong> </p>
                </div>
            </footer>
        </div>








    </section>

</template>

<script setup>



const sudoku = ref([])
const sudokuBkp = ref([])
const selectedCell = ref({})
const errors = ref(0)
const loading = ref(true)
const difficultySelect = ref('easy')
const difficulty = ref('')
const annotationMode = ref(false)


onMounted(async () => {
    loading.value = true
    if (localStorage.getItem('sudoku') === null) {
        await getSudoku()
    } else {
        sudoku.value = JSON.parse(localStorage.getItem('sudoku'))
        sudokuBkp.value = JSON.parse(localStorage.getItem('sudokuBkp'))
        difficulty.value = localStorage.getItem('difficulty')
    }

    loading.value = false

    difficultySelect.value = localStorage.getItem('difficulty')

    window.addEventListener("keydown", function (e) {
        if (selectedCell.value.editable == true) {

            selectedCell.value.oldNumber = selectedCell.value.number
            if (e.keyCode == 8 || e.keyCode == 46) {
                selectedCell.value.number = ''
            } else if (Number(e.key)) {

                if (annotationMode.value && e.key < 10) {
                    selectedCell.value.annotations[e.key - 1] = !selectedCell.value.annotations[e.key - 1]
                    localStorage.setItem('sudoku', JSON.stringify(response.data))
                    localStorage.setItem('sudokuBkp', JSON.stringify(response.data))
                    return
                }

                selectedCell.value.number = e.key
            }

            check()

        }
    });


})


function ehMarcada(cell) {

    if ((cell.row === selectedCell.value.row
        || cell.column === selectedCell.value.column
        || cell.block === selectedCell.value.block) && selectedCell) {

        return true
    }

    return false
}

async function getSudoku() {

    loading.value = true
    await $fetch(`https://sudoku-generator.onrender.com/api/v1/generate-${difficultySelect.value}/`, 'get')
        .then(response => {

            sudoku.value = response.data

            sudoku.value.forEach(cell => {
                cell['oldNumber'] = ''
                cell['conflicts'] = 0
                cell['annotations'] = [false, false, false, false, false, false, false, false, false]
                if (cell.number === '') {
                    cell['editable'] = true
                } else {
                    cell['editable'] = false
                }
                return
            })

            localStorage.setItem('sudoku', JSON.stringify(response.data))
            localStorage.setItem('sudokuBkp', JSON.stringify(response.data))
            localStorage.setItem('difficulty', difficultySelect.value)

            sudokuBkp.value = JSON.parse(JSON.stringify(sudoku.value))
            selectedCell.value = {}
            difficulty.value = difficultySelect.value
            loading.value = false
        })
    localStorage.setItem('difficulty', difficultySelect.value)
}

function reset() {

    sudoku.value = JSON.parse(JSON.stringify(sudokuBkp.value))

    localStorage.setItem('sudokuBkp', JSON.stringify(sudokuBkp.value))
    localStorage.setItem('sudoku', JSON.stringify(sudoku.value))

}

function addValue(event, valor) {
    selectedCell.value.oldNumber = selectedCell.value.number
    selectedCell.value.number = valor

    if(selectedCell.value.annotations[valor - 1] === true){
        selectedCell.value.annotations[valor - 1] = false
    }
    check()

}

function addAnnotation(event, index) {
    selectedCell.value.annotations[index] = !selectedCell.value.annotations[index]

    localStorage.setItem('sudokuBkp', JSON.stringify(sudokuBkp.value))
    localStorage.setItem('sudoku', JSON.stringify(sudoku.value))
}

function check() {

    sudoku.value.filter(cell => cell !== selectedCell.value).forEach(cell => {

        if ((cell.row === selectedCell.value.row
            || cell.column === selectedCell.value.column
            || cell.block === selectedCell.value.block)) {

            if (selectedCell.value.number == selectedCell.value.oldNumber || cell.number === '') {
                return
            }

            if (cell.number == selectedCell.value.number
                && selectedCell.value.number !== '') {
                cell.conflicts++
                selectedCell.value.conflicts++
                errors.value++

            }
            else if (selectedCell.value.oldNumber == cell.number) {
                cell.conflicts--
                selectedCell.value.conflicts--
                errors.value--

            }


        }
    })

    localStorage.setItem('sudoku', JSON.stringify(sudoku.value))

}

</script>

<style lang="scss" scoped>
#sudoku {

    position: relative;
    max-height: 100%;
    height: 100%;
    max-width: 100%;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    border: 3px solid #261D14;

}

#actions {
    height: max-content;
    margin: -10px
}

.button {
    border-radius: 5px;
}

.cell {
    position: relative;
    border: 1px solid #644F37;
    color: #261D14;
    width: calc(100% / 9);
    aspect-ratio: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    font-weight: 500;

}

#annotations {
    display: grid;
    position: absolute;
    padding: 2px;
    top: 0;
    left: 0;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    width: 100%;
    height: 100%;
    font-size: 12px;
}

.annotation {
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: hidden;
    visibility: hidden;
    border-radius: 50%;

    &.visible {
        visibility: visible !important;
    }
}

#difficulty-text {
    color: #261d14;
    font-weight: bold;
    display: flex;
    justify-content: center;

}

#difficulty-text {
    visibility: hidden;
    color: #261d14;
    display: flex;
    justify-content: center;

    &.visible {
        visibility: visible !important;
    }
}

.cell.editable {
    color: #261d14d5;

}

.cell::before {
    content: attr(data-number);
}

.cell:hover {
    background-color: rgba(173, 173, 173, 0.288);
}

.cell.selected {
    background-color: #F7D5A7;
    //color: black;
}

.cell.marcada:not(.selected) {
    background-color: rgba(247, 214, 167, 0.253) //color: black;
}

.cell.wrong:not(.editable) {
    background-color: #e76354 !important;
    color: white;
}

.cell.wrong {
    background-color: #e76354bb !important;
    color: white;
}

.cell[data-column="2"],
.cell[data-column="5"] {
    border-right: 3px solid #261d14da;
}

.cell[data-row="2"],
.cell[data-row="5"] {
    border-bottom: 3px solid #261d14da;
}


#loader {
    border: none;
    height: 55dvh;
    background-color: white;
    font-size: 100px;
    border-radius: 0;
}

#loader:before {
    content: 'Carregando Sudoku...';
    color: #261D14;
    font-weight: bold;
    position: absolute;
    font-size: 20px;
    top: 62%;
}

.navbar {
    background-color: #261D14 !important;
}



.navbar #logo-icon {
    bottom: 0;
    top: 1px;
    position: relative;
    color: #e76254;

    & img {
        border: 1px solid #E3D2BB;
        height: 1.5rem;

        border-radius: 50%;
    }
}

.navbar h1 {
    color: #E3D2BB !important;

}


.footer {
    background-color: #e3d2bbd5;;
}

#generate .button {
    background-color: #487545 !important;

}

#generate select {
    border-color: #487545 !important;
    background-color: white !important;
    color: #261D14;
    font-weight: bold;
}

#generate .select:after {

    border-color: #261D14;

}

#actions .button:not(#generate .button, #annotation-button, #erase-button, #reset-button) {
    background-color: #261d14ef;
    border: #261d14 2px solid;
    color: #E3D2BB;
    box-shadow: #261d14b7 2px 2px 3px;
}


#erase-button {
    background-color: #e76254;
    border: #e76254 2px solid;
    box-shadow: #261d14b7 2px 2px 3px;
    color: #ffffff;

    &:hover {
        background-color: #c43e2f;
    }

}

#reset-button {
    background-color: #9C8B68;
    border: #9C8B68 2px solid;
    box-shadow: #261d14b7 2px 2px 3px;
    color: #ffffff;

    &:hover {
        background-color: #836e43;
    }

}



#annotation-button {

    background-color: white;
    border: #261d14 1px solid;
    color: #261D14;
    box-shadow: #261d14b7 2px 2px 3px;

}

#annotation-button.is-active {
    background-color: #487545;
    border-color: #487545;
    color: white;
    box-shadow: #261d14b7 2px 2px 3px;

}

.hero, .hero-body{
    background-color: white !important;
}


@media screen and (max-width: 768px) {
    #actions .button.is-large {
        aspect-ratio: 1;
        font-size: 1rem;

    }

    #actions .column {
        padding-top: 0.5rem !important;
        padding-bottom: 0.5rem !important;
        padding-left: 0px;
    }

    .hero-body {
        padding: 0.75rem !important;

        & .column:has(#actions) {
            padding-top: 0rem !important;
        }

    }

    #loader {
        font-size: 50px !important;
        height: 50.9dvh !important;
    }

    #annotations {
        font-size: 11px !important;
    }

}
</style>