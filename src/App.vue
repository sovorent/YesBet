<template>
<div id="app" class="container">
    <div class="page-header text-center" style="padding-bottom:20px;">
        <h1>YES + BET<br/> <small>You can Yes. You can Bet.</small></h1>
    </div>
    <div id="Matchs" v-for="match in Matchs" style="padding-bottom:20px;">
        <div class="card " v-bind:class="{ tran: isDisabled(match.status)}">
            <div class="card-header">
                {{match.matchName}}
            </div>
            <div class="row">
                <div class="card-body col-md-6 ">
                    <h5 class="card-title">Team {{match.first}}</h5>
                    <div class="input-group mb-3">
                        <vue-numeric class="form-control" v-bind:precision="3" separator="," v-model="match.betPrice0" :disabled="isDisabled(match.status)"></vue-numeric>
                        <div class="input-group-append">
                            <button class="btn btn-outline-danger" type="button" :disabled="isDisabled(match.status)" v-on:click="letBet(match.contractAddress, 0, match.betPrice0)" style="width:100px;">
                            Bet</button>
                        </div>
                    </div>
                </div>
                <div class="card-body col-md-6 text-right">
                    <h5 class="card-title">Team {{match.secound}}</h5>
                    <div class="input-group mb-3">
                        <div class="input-group-prepend">
                            <button class="btn btn-outline-primary" type="button" :disabled="isDisabled(match.status)" v-on:click="letBet(match.contractAddress, 1, match.betPrice1)" style="width:100px;">
                              Bet</button>
                        </div>
                        <vue-numeric class="form-control" v-bind:precision="3" separator="," v-model="match.betPrice1" :disabled="isDisabled(match.status)"></vue-numeric>
                    </div>
                </div>
            </div>
            <div class="row text-center">
                <div class="col-md-12">
                    Balance = {{match.matchBalance}}
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<script>
import Firebase from 'firebase'
import toastr from 'toastr'
import Web3 from 'web3'
import Vue from 'vue'
import VueNumeric from 'vue-numeric'

Vue.use(VueNumeric)
// Initializing
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
}
// Get default address
web3.eth.defaultAccount = web3.eth.accounts[0];
var abi = [{
        "constant": false,
        "inputs": [],
        "name": "Ownable",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "constant": true,
        "inputs": [],
        "name": "owner",
        "outputs": [{
            "name": "",
            "type": "address"
        }],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
    },
    {
        "inputs": [{
                "name": "_betName",
                "type": "string"
            },
            {
                "name": "_leftSideName",
                "type": "string"
            },
            {
                "name": "_rightSideName",
                "type": "string"
            }
        ],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "constructor"
    },
    {
        "anonymous": false,
        "inputs": [{
                "indexed": true,
                "name": "previousOwner",
                "type": "address"
            },
            {
                "indexed": true,
                "name": "newOwner",
                "type": "address"
            }
        ],
        "name": "OwnershipTransferred",
        "type": "event"
    },
    {
        "constant": false,
        "inputs": [{
            "name": "_side",
            "type": "bool"
        }],
        "name": "betting",
        "outputs": [],
        "payable": true,
        "stateMutability": "payable",
        "type": "function"
    },
    {
        "constant": false,
        "inputs": [{
            "name": "_winner",
            "type": "bool"
        }],
        "name": "payToWinner",
        "outputs": [],
        "payable": true,
        "stateMutability": "payable",
        "type": "function"
    },
    {
        "constant": true,
        "inputs": [],
        "name": "getMatch",
        "outputs": [{
                "name": "_name",
                "type": "string"
            },
            {
                "name": "_leftSide",
                "type": "string"
            },
            {
                "name": "_rightSide",
                "type": "string"
            }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
    },
    {
        "constant": true,
        "inputs": [],
        "name": "getBalance",
        "outputs": [{
            "name": "_balance",
            "type": "uint256"
        }],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
    },
    {
        "constant": true,
        "inputs": [],
        "name": "getPool",
        "outputs": [{
                "name": "_name",
                "type": "string"
            },
            {
                "name": "_leftSide",
                "type": "string"
            },
            {
                "name": "_rightSide",
                "type": "string"
            },
            {
                "name": "_amountA",
                "type": "uint256"
            },
            {
                "name": "_amountB",
                "type": "uint256"
            },
            {
                "name": "_lenA",
                "type": "uint256"
            },
            {
                "name": "_lenB",
                "type": "uint256"
            }
        ],
        "payable": false,
        "stateMutability": "view",
        "type": "function"
    }
];

let config = {
    apiKey: "AIzaSyANO56rF2l1pszEOERr8zt61Gn0GDJ1UeU",
    authDomain: "yesbet-8d794.firebaseapp.com",
    databaseURL: "https://yesbet-8d794.firebaseio.com",
    projectId: "yesbet-8d794",
    storageBucket: "yesbet-8d794.appspot.com",
    messagingSenderId: "597788857277"
};

let app = Firebase.initializeApp(config)
let db = app.database()
let booksRef = db.ref('matchDetail')
var dataList = [];
var data = {};
booksRef.on('value', function (Snapshot) {
    Snapshot.forEach(function (Snap) {
        var obj = Snap.val();
        var contractAddress = obj.contractAddress;
        var CoursetroContract = web3.eth.contract(abi);
        var Coursetro = CoursetroContract.at(contractAddress);
        Coursetro.getMatch(function (err, result) {
            if (err) alert(err);
            var data = {};
            data.id = Snap.val().id;
            data.contractAddress = Snap.val().contractAddress;
            data.status = Snap.val().status;
            data.create = Snap.val().create;
            data.matchName = result[0];
            data.first = result[1];
            data.secound = result[2];
            data.matchBalance = Coursetro.getBalance(function (error, result) {
                if (!error) {
                    console.log(result.toNumber());
                } else {
                    return result.toNumber();
                }
            });
            dataList.push(data);

        });
    })
});

function getBalanceMatch(matchAddress) {
    var contractAddress = matchAddress;
    var CoursetroContract = web3.eth.contract(abi);
    var Coursetro = CoursetroContract.at(contractAddress);
    Coursetro.getBalance(function (error, result) {
        if (!error) {
            console.log(result.toNumber());
            return result.toNumber();
        } else
            console.error(error);
    });
}

export default {
    name: 'app',
    data: () => ({
        Matchs: dataList
    }),
    methods: {
        isDisabled: function (status) {
            return status == 'pause' ? true : false;
        },
        letBet: function (matchAdress, team, betPrice) {

            if (betPrice > 0) {
                var contractAddress = matchAdress;
                var abiContract = web3.eth.contract(abi);
                var betContract = abiContract.at(contractAddress);
                betContract.betting(team, {
                    from: web3.eth.defaultAccount,
                    value: web3.toWei(betPrice, 'ether')
                }, function (err, transactionHash) {
                    if (err) {
                        console.log(err);
                        return;
                    }
                    window.open('https://kovan.etherscan.io/tx/' + transactionHash);
                })
            }
        }
    },
    computed: {
        MatchsSort: function () {
            return _.orderBy(this.Matchs, 'id')
        }
    }
}
</script>

<style>
body {}

#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    margin-top: 20px;
}

.card {
    --background-color: #e9e4e4;
    background-image: url('../img/cardsbg.png');
    background-position: center;
}

.card-header {
    color: white;
    font-weight: bold;
    height: 48px;
    background-image: url('../img/title.jpg');
    background-position: center;
    background-repeat: no-repeat;
}

.tran {
    -ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=50)";
    /* IE 8 */
    filter: alpha(opacity=50);
    /* IE 5-7 */
    -moz-opacity: 0.5;
    /* Netscape */
    -khtml-opacity: 0.5;
    /* Safari 1.x */
    opacity: 0.5;
    /* Good browsers */
}
</style>
