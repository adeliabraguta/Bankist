<script setup>
import {computed, onMounted, ref} from "vue";
// Data
const account1 = {
    owner: 'Jonas Schmedtmann',
    movements: [200, 450, -400, 3000, -650, -130, 70, 1300],
    interestRate: 1.2, // %
    pin: 1111,
};

const account2 = {
    owner: 'Jessica Davis',
    movements: [5000, 3400, -150, -790, -3210, -1000, 8500, -30],
    interestRate: 1.5,
    pin: 2222,
};

const account3 = {
    owner: 'Steven Thomas Williams',
    movements: [200, -200, 340, -300, -20, 50, 400, -460],
    interestRate: 0.7,
    pin: 3333,
};

const account4 = {
    owner: 'Sarah Smith',
    movements: [430, 1000, 700, 50, 90],
    interestRate: 1,
    pin: 4444,
};

const accounts = [account1, account2, account3, account4];

/////////////////////////////////////////////////

const currencies = new Map([
    ['USD', 'United States dollar'],
    ['EUR', 'Euro'],
    ['GBP', 'Pound sterling'],
]);

//username
const createUserNames = function (accs) {
    accs.forEach(function (acc) {
        acc.username = acc.owner.toLowerCase().split(' ').map(name => name[0]).join('')
    })
}
createUserNames(accounts)


//balance
let formattedBalance = ref('')
const displayBalance = function (acc) {
    acc.balance = acc.movements.reduce((acc, mov) => acc + mov, 0);
    formattedBalance = `${acc.balance}€`
};

//summary
let labelSumIn = ref('')
let labelSumOut = ref('')
let labelSumInterest = ref('')

const displaySummery = function (acc) {
    const incomes = acc.movements.filter(mov => mov > 0).reduce((acc, curr) => acc + curr)
    labelSumIn = `${incomes}€`

    const outcomes = Math.abs(acc.movements.filter(mov => mov < 0).reduce((acc, curr) => acc + curr))
    labelSumOut= `${outcomes}€`

    const interest = acc.movements.filter(mov => mov > 0).map(dep => (dep * 1.2) / 100).reduce((dep, curr) => dep + curr).toFixed(2)
    labelSumInterest= `${interest}€`

}


let currentAccount = ref(null);
let labelWelcome = ref('Log in to get started')
const containerApp = ref(null)
const inputLoginUsername = ref('')
const inputLoginPin = ref('')
const btnLogin = (e) => {
    e.preventDefault();
    currentAccount.value  = accounts.find(acc => acc.username === inputLoginUsername.value);
    if (currentAccount.value?.pin === Number(inputLoginPin.value)) {
        labelWelcome =  `Welcome back, ${currentAccount.value.owner.split(' ')[0]}`
        containerApp.value.style.opacity = 100;
        inputLoginPin.value = inputLoginUsername.value = '';
        displayBalance(currentAccount.value);
        displaySummery(currentAccount.value)
    }
}

const receiveAcc =ref(null)
const inputTransferTo = ref('')
const inputTransferAmount = ref('')
const btnTransfer = (e) => {
    e.preventDefault();
    receiveAcc.value = accounts.find(acc => acc.username === inputTransferTo.value)
    const amount = Number(inputTransferAmount.value)
    inputTransferTo.value = inputTransferAmount.value = ''
    if(amount > 0 && receiveAcc && currentAccount.value.balance >=0 && receiveAcc?.value.username !== currentAccount.value.username){
        currentAccount.value.movements.push(-amount)
        receiveAcc.value.movements.push(amount)
        displaySummery(currentAccount.value)
        displayBalance(currentAccount.value)
    }

}

</script>

<template>
    <!-- TOP NAVIGATION -->
    <nav>
        <p class="welcome"> {{labelWelcome}}</p>
        <img src="../assets/logo.png" alt="Logo" class="logo"/>
        <form class="login">
            <input
                    type="text"
                    placeholder="user"
                    class="login__input login__input--user"
                    v-model="inputLoginUsername"
            />
            <!-- In practice, use type="password" -->
            <input
                    type="text"
                    placeholder="PIN"
                    maxlength="4"
                    class="login__input login__input--pin"
                    v-model="inputLoginPin"
            />
            <button class="login__btn" @click="btnLogin" > &rarr;</button>
        </form>
    </nav>

    <main class="app" ref="containerApp">
        <!-- BALANCE -->
        <div class="balance">
            <div>
                <p class="balance__label">Current balance</p>
                <p class="balance__date">
                    As of <span class="date">05/03/2037</span>
                </p>
            </div>
            <p class="balance__value" >{{formattedBalance}}</p>
        </div>

        <!-- MOVEMENTS -->
        <div class="movements">
        <div v-for="(mov, i) in currentAccount?.movements.slice().reverse()" :key="i" class="movements__row">
            <div class="movements__type" :class="'movements__type--' + (mov > 0 ? 'deposit' : 'withdrawal')">
                {{ currentAccount.movements.length - i }} {{ mov > 0 ? 'deposit' : 'withdrawal' }}
            </div>
            <div class="movements__date">3 days ago</div>
            <div class="movements__value">{{ mov }}€</div>
        </div>
    </div>

        <!-- SUMMARY -->
        <div class="summary">
            <p class="summary__label">In</p>
            <p class="summary__value summary__value--in">{{labelSumIn}}</p>
            <p class="summary__label">Out</p>
            <p class="summary__value summary__value--out">{{labelSumOut}}</p>
            <p class="summary__label">Interest</p>
            <p class="summary__value summary__value--interest">{{labelSumInterest}}</p>
            <button class="btn--sort">&downarrow; SORT</button>
        </div>

        <!-- OPERATION: TRANSFERS -->
        <div class="operation operation--transfer">
            <h2>Transfer money</h2>
            <form class="form form--transfer">
                <input type="text" class="form__input form__input--to" v-model="inputTransferTo"/>
                <input type="number" class="form__input form__input--amount" v-model="inputTransferAmount"/>
                <button class="form__btn form__btn--transfer" @click="btnTransfer">&rarr;</button>
                <label class="form__label">Transfer to</label>
                <label class="form__label">Amount</label>
            </form>
        </div>

        <!-- OPERATION: LOAN -->
        <div class="operation operation--loan">
            <h2>Request loan</h2>
            <form class="form form--loan">
                <input type="number" class="form__input form__input--loan-amount"/>
                <button class="form__btn form__btn--loan">&rarr;</button>
                <label class="form__label form__label--loan">Amount</label>
            </form>
        </div>

        <!-- OPERATION: CLOSE -->
        <div class="operation operation--close">
            <h2>Close account</h2>
            <form class="form form--close">
                <input type="text" class="form__input form__input--user"/>
                <input
                        type="password"
                        maxlength="6"
                        class="form__input form__input--pin"
                />
                <button class="form__btn form__btn--close">&rarr;</button>
                <label class="form__label">Confirm user</label>
                <label class="form__label">Confirm PIN</label>
            </form>
        </div>

        <!-- LOGOUT TIMER -->
        <p class="logout-timer">
            You will be logged out in <span class="timer">05:00</span>
        </p>
    </main>

    <!-- <footer>
      &copy; by Jonas Schmedtmann. Don't claim as your own :)
    </footer> -->

</template>

<style scoped>
/*
 * Use this CSS to learn some intersting techniques,
 * in case you're wondering how I built the UI.
 * Have fun! 😁
 */

* {
    margin: 0;
    padding: 0;
    box-sizing: inherit;
}

html {
    font-size: 62.5%;
    box-sizing: border-box;
}

body {
    font-family: "Poppins", sans-serif;
    color: #444;
    background-color: #f3f3f3;
    height: 100vh;
    padding: 2rem;
}

nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 2rem;
}

.welcome {
    font-size: 1.9rem;
    font-weight: 500;
}

.logo {
    height: 5.25rem;
}

.login {
    display: flex;
}

.login__input {
    border: none;
    padding: 0.5rem 2rem;
    font-size: 1.6rem;
    font-family: inherit;
    text-align: center;
    width: 12rem;
    border-radius: 10rem;
    margin-right: 1rem;
    color: inherit;
    border: 1px solid #fff;
    transition: all 0.3s;
}

.login__input:focus {
    outline: none;
    border: 1px solid #ccc;
}

.login__input::placeholder {
    color: #bbb;
}

.login__btn {
    border: none;
    background: none;
    font-size: 2.2rem;
    color: inherit;
    cursor: pointer;
    transition: all 0.3s;
}

.login__btn:hover,
.login__btn:focus,
.btn--sort:hover,
.btn--sort:focus {
    outline: none;
    color: #777;
}

/* MAIN */
.app {
    position: relative;
    max-width: 100rem;
    margin: 4rem auto;
    display: grid;
    grid-template-columns: 4fr 3fr;
    grid-template-rows: auto repeat(3, 15rem) auto;
    gap: 2rem;
    opacity: 0;
    /* NOTE This creates the fade in/out anumation */
    transition: all 1s;
}

.balance {
    grid-column: 1 / span 2;
    display: flex;
    align-items: flex-end;
    justify-content: space-between;
    margin-bottom: 2rem;
}

.balance__label {
    font-size: 2.2rem;
    font-weight: 500;
    margin-bottom: -0.2rem;
}

.balance__date {
    font-size: 1.4rem;
    color: #888;
}

.balance__value {
    font-size: 4.5rem;
    font-weight: 400;
}

/* MOVEMENTS */
.movements {
    grid-row: 2 / span 3;
    background-color: #fff;
    border-radius: 1rem;
    overflow: scroll;
}

.movements__row {
    padding: 2.25rem 4rem;
    display: flex;
    align-items: center;
    border-bottom: 1px solid #eee;
}

.movements__type {
    font-size: 1.1rem;
    text-transform: uppercase;
    font-weight: 500;
    color: #fff;
    padding: 0.1rem 1rem;
    border-radius: 10rem;
    margin-right: 2rem;
}

.movements__date {
    font-size: 1.1rem;
    text-transform: uppercase;
    font-weight: 500;
    color: #666;
}

.movements__type--deposit {
    background-image: linear-gradient(to top left, #39b385, #9be15d);
}

.movements__type--withdrawal {
    background-image: linear-gradient(to top left, #e52a5a, #ff585f);
}

.movements__value {
    font-size: 1.7rem;
    margin-left: auto;
}

/* SUMMARY */
.summary {
    grid-row: 5 / 6;
    display: flex;
    align-items: baseline;
    padding: 0 0.3rem;
    margin-top: 1rem;
}

.summary__label {
    font-size: 1.2rem;
    font-weight: 500;
    text-transform: uppercase;
    margin-right: 0.8rem;
}

.summary__value {
    font-size: 2.2rem;
    margin-right: 2.5rem;
}

.summary__value--in,
.summary__value--interest {
    color: #66c873;
}

.summary__value--out {
    color: #f5465d;
}

.btn--sort {
    margin-left: auto;
    border: none;
    background: none;
    font-size: 1.3rem;
    font-weight: 500;
    cursor: pointer;
}

/* OPERATIONS */
.operation {
    border-radius: 1rem;
    padding: 3rem 4rem;
    color: #333;
}

.operation--transfer {
    background-image: linear-gradient(to top left, #ffb003, #ffcb03);
}

.operation--loan {
    background-image: linear-gradient(to top left, #39b385, #9be15d);
}

.operation--close {
    background-image: linear-gradient(to top left, #e52a5a, #ff585f);
}

h2 {
    margin-bottom: 1.5rem;
    font-size: 1.7rem;
    font-weight: 600;
    color: #333;
}

.form {
    display: grid;
    grid-template-columns: 2.5fr 2.5fr 1fr;
    grid-template-rows: auto auto;
    gap: 0.4rem 1rem;
}

/* Exceptions for interst */
.form.form--loan {
    grid-template-columns: 2.5fr 1fr 2.5fr;
}

.form__label--loan {
    grid-row: 2;
}

/* End exceptions */

.form__input {
    width: 100%;
    border: none;
    background-color: rgba(255, 255, 255, 0.4);
    font-family: inherit;
    font-size: 1.5rem;
    text-align: center;
    color: #333;
    padding: 0.3rem 1rem;
    border-radius: 0.7rem;
    transition: all 0.3s;
}

.form__input:focus {
    outline: none;
    background-color: rgba(255, 255, 255, 0.6);
}

.form__label {
    font-size: 1.3rem;
    text-align: center;
}

.form__btn {
    border: none;
    border-radius: 0.7rem;
    font-size: 1.8rem;
    background-color: #fff;
    cursor: pointer;
    transition: all 0.3s;
}

.form__btn:focus {
    outline: none;
    background-color: rgba(255, 255, 255, 0.8);
}

.logout-timer {
    padding: 0 0.3rem;
    margin-top: 1.9rem;
    text-align: right;
    font-size: 1.25rem;
}

.timer {
    font-weight: 600;
}

</style>
