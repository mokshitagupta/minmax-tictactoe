<script>
    import Counter from "./lib/Counter.svelte";

    // 0 -> O, 1 -> X
    let turn = $state(0);
    let play = $state(0);
    let winner = $state(null);
    let board = $state(Array(3).fill(Array(3).fill(-1)));
    let AI_OP = 1;

    function reset() {
        board = Array(3).fill(Array(3).fill(-1));
        play++;
        turn = 0;
        winner = null;
    }

    function getStates(board, turn) {
        // console.log(board)
        let ret = [];
        for (let i = 0; i < board.length; i++) {
            for (let j = 0; j < board.length; j++) {
                if (board[i][j] < 0) {
                    let s = board.map((e) => Array.from(e));
                    s[i][j] = turn;
                    ret.push(s);
                }
            }
        }
        return ret;
    }

    function indexOfMin(arr) {
        if (arr.length === 0) {
            return -1; // Handle empty array
        }

        let minIndex = 0;
        let minValue = arr[0];

        for (let i = 1; i < arr.length; i++) {
            if (arr[i] < minValue) {
                minValue = arr[i];
                minIndex = i;
            }
        }

        return minIndex;
    }

    function getStateInds(board) {
        let ret = [];
        for (let i = 0; i < board.length; i++) {
            for (let j = 0; j < board.length; j++) {
                if (board[i][j] < 0) {
                    ret.push([i, j]);
                }
            }
        }
        return ret;
    }

    let ct = 0;
    let seen = {};

    // function

    function getScore(winr, maxi, board, d) {
        let sqs = board.flat().filter((el) => el === -1).length;
        // let extra = checkBlocked(board)
        let extra = 0
        // console.log(sqs)
        if (winr < 0) {
            return 0;
        }

        if (winr === 0) {
            return extra + (10 * (sqs + 1) - d);
        }

        if (winr === 1) {
            return - extra + d - (10 * (sqs + 1));
        }
    }

    function checkRecursive(board, maxi, d) {
        // console.log(board)
        // if (board.flat().includes(undefined)){
        //     console.log(board , "NOOOOOOOOOOOOOOOOO11111111111")
        // }

        console.log(maxi)
        let winr = checkWinner(board);
        d++;
        if (winr != null) {
            // console.log(winr, maxi, board, getScore(winr,maxi))
            return getScore(winr, maxi, board, d);
        } else {
            // look ahead
        }

        if (maxi === true) {
            let maxscore = -Infinity;
            let states = getStates(board, 1);
            // console.log(states)
            for (let s of states) {
                // if (board.flat().includes(undefined)){
                //     console.log(board, "NOOOOOOOOOOOOOOOOO11111111111")
                // }
                let evalu = checkRecursive(s, false, d);
                // console.log(evalu)
                maxscore = Math.max(maxscore, evalu);
            }
            return maxscore;
        } else {
            let minscore = Infinity;
            let states = getStates(board, 0);
            for (let s of states) {
                // if (board.flat().includes(undefined)){
                //     console.log(board, "NOOOOOOOOOOOOOOOOO11111111111")
                // }
                let evalu = checkRecursive(s, true, d);
                // console.log(evalu)
                minscore = Math.min(minscore, evalu);
            }
            return minscore;
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
                // console.log(els.values().next().value, "wins!");
                return els.values().next().value;
            }
            els.clear();
            for (let j = 0; j < board.length; j++) {
                els.add(board[j][i]);
            }
            if (els.size === 1 && els.values().next().value >= 0) {
                // console.log(els.values().next().value, "wins!");
                return els.values().next().value;
            }
        }

        // console.log(el, el1);
        if (el.size === 1 && el.values().next().value >= 0) {
            // console.log(el.values().next().value, "wins!");
            return el.values().next().value;
        }

        if (el1.size === 1 && el1.values().next().value >= 0) {
            // console.log(el1.values().next().value, "wins!");
            return el1.values().next().value;
        }
        if (new Set(board.flat()).has(-1) === false) {
            // console.log("players tied!");
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
                            if (winner !== null){
                                return
                            }
                            let states = getStates(board, turn);
                            let inds = getStateInds(board);
                            let results = [];
                            console.log("TURN ==>", turn);

                            if (turn === AI_OP && (board.flat().filter(e => (e === -1)).length > 0)) {
                                for (let i = 0; i < states.length; i++) {
                                    console.log("MAIN RECURSE", states[i]);
                                    let score = checkRecursive(
                                        states[i],
                                        false,
                                        0,
                                    );
                                    // console.log(score)
                                    results.push(score);
                                }
                                console.log(results, states);
                                let ind = inds[indexOfMin(results)];
                                board[ind[0]][ind[1]] = 1;
                                let action = document.getElementById(
                                    `cell-${ind[0]}-${ind[1]}`,
                                );
                                action.innerText = "X";
                                action.disabled = true;
                                turn = (turn + 1) % 2;
                                winner = checkWinner(board);
                            }
                            // console.log(checkRecursive(board, false))
                            // console.log(board)
                            // console.log(states)
                            // console.log(winner)
                        }}
                        id={`cell-${row}-${col}`}
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
                    ? "Players tied!"
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
