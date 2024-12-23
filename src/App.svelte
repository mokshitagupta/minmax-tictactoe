<script>
    import Counter from "./lib/Counter.svelte";

    // 0 -> O, 1 -> X
    let turn = $state(0);
    let play = $state(0);
    let winner = $state(null);
    let board = $state(Array(3).fill(Array(3).fill(-1)));

    function reset() {
        board = Array(3).fill(Array(3).fill(-1));
        play++;
        turn = 0;
        winner = null;
    }

    function getStates(board, turn) {
        // console.log(board)
        let myb = board
        let ret = []
        for (let i = 0; i < myb.length; i++){
            for (let j = 0; j < myb.length; j++){
                if (myb[i][j] < 0){
                    let s = board.map(e => Array.from(e))
                    s[i][j] = turn
                    ret.push(s)
                }
            }
        }
        return ret
    }

    let ct = 0
    let seen = {}

    function checkRecursive(board, turn, orgTurn){
        let winr = null
        ct ++
        // console.log("recurse", ct)
        while (winr === null) {

            if (board.flat().join("") in seen){
                return seen[board.flat().join("")]
            }
            winr = checkWinner(board)
            // console.log("curr winner", winr, turn)
            if (winr === null) {
                let states = getStates(board, (turn + 1) % 2)
                // console.log("More")
                let ret = []
                for (let s of states){
                    // let res = 
                    // console.log("checking", s, (turn + 2) % 2)
                    ret.push(checkRecursive(s, (turn + 1) % 2, orgTurn))
                }
                // console.log("result of iteration", ret, ct)

                let r = Math.max(...ret)
                seen[board.flat().join("")] = r
                return r
                // return
            } else {
                // console.log("Terminal")
                if (winr === orgTurn){
                    seen[board.flat().join("")] = -10
                    console.log("ret -10", winr, turn, orgTurn)
                    return -10
                } else if (winr < 0) {
                    console.log("ret 0", winr, turn, orgTurn)
                    seen[board.flat().join("")] = 0
                    return 0
                } else {
                    console.log("ret 10", winr, turn, orgTurn)
                    seen[board.flat().join("")] = 10
                    return 10
                }
            } 
        }
    }

    function checkWinner(board) {
        // console.log("check")
        let el = new Set();
        let el1 = new Set();
        for (let i = 0; i < board.length; i++) {
            el.add(board[i][i]);
            el1.add(board[i][board.length - 1 - i]);
            let els = new Set(board[i]);
            if (els.size === 1 && els.values().next().value >= 0) {
                console.log(els.values().next().value, "wins!");
                return els.values().next().value;
            }
            els.clear();
            for (let j = 0; j < board.length; j++) {
                els.add(board[j][i]);
            }
            if (els.size === 1 && els.values().next().value >= 0) {
                console.log(els.values().next().value, "wins!");
                return els.values().next().value;
            }
        }

        // console.log(el, el1);
        if (el.size === 1 && el.values().next().value >= 0) {
            console.log(el.values().next().value, "wins!");
            return el.values().next().value;
        }

        if (el1.size === 1 && el1.values().next().value >= 0) {
            console.log(el1.values().next().value, "wins!");
            return el1.values().next().value;
        }
        if (new Set(board.flat()).has(-1) === false) {
            console.log("players tied!");
            return -1;
        }

        return null;
    }
</script>

<main>
    <button id="reset" onclick={reset}> Reset Board </button>
    {#key play}
    <div id="tgrid">
            {#each { length: 3 }, row}
                {#each { length: 3 }, col}
                    <button
                        data-col={col}
                        data-row={row}
                        aria-label="cell"
                        onclick={(e) => {
                            let c = parseInt(e.target.dataset.col);
                            let r = parseInt(e.target.dataset.row);
                            board[r][c] = turn;
                            e.target.innerText = turn == 0 ? "O" : "X";
                            e.target.disabled = true;
                            turn = (turn + 1) % 2;
                            winner = checkWinner(board);
                            let states = getStates(board, turn)
                            let results = []
                            console.log("TURN ==>", turn)
                            for (let i = 0; i < states.length; i++){
                                console.log("MAIN RECURSE", states[i])
                                let score = checkRecursive(states[i], turn, turn)
                                // console.log(score)
                                results.push(score)
                            }
                            console.log(results, states)
                            // console.log(board)
                            // console.log(states)
                            // console.log(winner)
                        }}
                        class="cell"
                    >
                    </button>
                {/each}
            {/each}
        </div>
    {/key}
    {#key winner}
    {#if winner !== null}
        <p id="win">
            {winner === -1
                ? "Players ied!"
                : `Player ${winner === 0 ? "O" : "X"} wins!`}
        </p>
    {:else}
    <p id="win"></p>
    {/if}
{/key}
    
</main>

<style>
    :root {
        --width: 100px;
    }

    main {
        height: 100vh;
        width: 100vw;
        display: flex;
        flex-direction: column;
        gap: 20px;
        justify-content: center;
        align-items: center;
    }

    #win {
        min-height: 20px;
    }

    #reset {
        background-color: cadetblue;
        border-width: 0px;
        transition: all 0.7s cubic-bezier(0.19, 1, 0.22, 1);
    }

    #reset:hover {
        background-color: rgb(64, 129, 131);
    }

    .cell {
        width: var(--width);
        height: var(--width);
        /* display: block; */
        background-color: white;
        color: black;
        font-size: 15px;
        /* position: relative; */
    }

    #tgrid {
        width: calc(var(--width) * 3.2);
        height: calc(var(--width) * 3.2);
        display: grid;
        /* position: absolute; */
        grid-template-rows: repeat(3, 1fr);
        grid-template-columns: repeat(3, 1fr);
        /* gap: 5px; */
    }
</style>
