<template>
  <section>
    <button
      class="btn btn-primary btn-lg btn-block mb-3"
      id="connectButton"
      disabled
    >
      Connect
    </button>
    <!-- <button class="btn btn-primary btn-lg btn-block mb-3" id="getAccounts">
      eth_accounts
    </button> -->
    <p v-show="address" class="info-text alert alert-secondary">
      Connected address: {{ address }}
    </p>
  </section>
</template>
<script>
const Web3 = require("web3");
const forwarderOrigin = "http://localhost:9010";
let web3 = new Web3(Web3.givenProvider || "ws://localhost:9010");
//console.log(web3.utils.toChecksumAddress("0x8762db106b2c2a0bccb3a80d1ed41273552616e8"));
const abi = [
  {
    constant: true,
    inputs: [],
    name: "name",
    outputs: [
      {
        name: "",
        type: "string",
      },
    ],
    payable: false,
    stateMutability: "view",
    type: "function",
  },
  {
    constant: false,
    inputs: [
      {
        name: "_spender",
        type: "address",
      },
      {
        name: "_value",
        type: "uint256",
      },
    ],
    name: "approve",
    outputs: [
      {
        name: "",
        type: "bool",
      },
    ],
    payable: false,
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    constant: true,
    inputs: [],
    name: "totalSupply",
    outputs: [
      {
        name: "",
        type: "uint256",
      },
    ],
    payable: false,
    stateMutability: "view",
    type: "function",
  },
  {
    constant: false,
    inputs: [
      {
        name: "_from",
        type: "address",
      },
      {
        name: "_to",
        type: "address",
      },
      {
        name: "_value",
        type: "uint256",
      },
    ],
    name: "transferFrom",
    outputs: [
      {
        name: "",
        type: "bool",
      },
    ],
    payable: false,
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    constant: true,
    inputs: [],
    name: "decimals",
    outputs: [
      {
        name: "",
        type: "uint8",
      },
    ],
    payable: false,
    stateMutability: "view",
    type: "function",
  },
  {
    constant: true,
    inputs: [
      {
        name: "_owner",
        type: "address",
      },
    ],
    name: "balanceOf",
    outputs: [
      {
        name: "balance",
        type: "uint256",
      },
    ],
    payable: false,
    stateMutability: "view",
    type: "function",
  },
  {
    constant: true,
    inputs: [],
    name: "symbol",
    outputs: [
      {
        name: "",
        type: "string",
      },
    ],
    payable: false,
    stateMutability: "view",
    type: "function",
  },
  {
    constant: false,
    inputs: [
      {
        name: "_to",
        type: "address",
      },
      {
        name: "_value",
        type: "uint256",
      },
    ],
    name: "transfer",
    outputs: [
      {
        name: "",
        type: "bool",
      },
    ],
    payable: false,
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    constant: true,
    inputs: [
      {
        name: "_owner",
        type: "address",
      },
      {
        name: "_spender",
        type: "address",
      },
    ],
    name: "allowance",
    outputs: [
      {
        name: "",
        type: "uint256",
      },
    ],
    payable: false,
    stateMutability: "view",
    type: "function",
  },
  {
    payable: true,
    stateMutability: "payable",
    type: "fallback",
  },
  {
    anonymous: false,
    inputs: [
      {
        indexed: true,
        name: "owner",
        type: "address",
      },
      {
        indexed: true,
        name: "spender",
        type: "address",
      },
      {
        indexed: false,
        name: "value",
        type: "uint256",
      },
    ],
    name: "Approval",
    type: "event",
  },
  {
    anonymous: false,
    inputs: [
      {
        indexed: true,
        name: "from",
        type: "address",
      },
      {
        indexed: true,
        name: "to",
        type: "address",
      },
      {
        indexed: false,
        name: "value",
        type: "uint256",
      },
    ],
    name: "Transfer",
    type: "event",
  },
];

//window.addEventListener("DOMContentLoaded", initialize);
module.exports = {
  name: "SignInMM",
  mounted() {
    this.initialize();
  },
  data() {
    return {
      tokenInfo: null,
      address: null,
    };
  },
  methods: {
    setData(address, info) {
      this.address = address;
      this.tokenInfo = info;
      console.log(address);
      console.log(info);
      const data = {
        address: this.address,
        tokenInfo: this.tokenInfo,
      };
      this.$emit("set-info", data);
    },
    parsingWallet() {
      this.$emit("parsing-wallet");
    },
    initialize: async function () {
      let accounts;
      let tokenInfo = {};
      let tokenPrices;
      const isMetaMaskConnected = () => accounts && accounts.length > 0;
      const onboardButton = document.getElementById("connectButton");
      const getAccountsButton = document.getElementById("getAccounts");
      const getAccountsResult = document.getElementById("getAccountsResult");
      const tokendisplay = document.getElementById("tokendisplay");

      const isMetaMaskInstalled = () => {
        // Must check ethereum binding on window object to see if installed
        const { ethereum } = window;
        return Boolean(ethereum && ethereum.isMetaMask);
      };

      // Parse tokens from address transaction json
      const parseTokens = async (data) => {
        let omit = {};
        for (let i = 0; i < data.length; i++) {
          const { contractAddress, gasUsed, tokenName, tokenSymbol } = data[i];

          // Add new token to info object
          if (tokenInfo[contractAddress] == null && !omit[contractAddress]) {
            let checkSumAddress = web3.utils.toChecksumAddress(contractAddress);
            let tokenInst = new web3.eth.Contract(abi, contractAddress);
            let balance = await tokenInst.methods.balanceOf(accounts[0]).call();

            if (balance > 0) {
              let dec = await tokenInst.methods.decimals().call();
              let newbal = balance / Math.pow(10, dec);

              let obj = {};
              obj.contract = contractAddress;
              obj.checkSumAddress = checkSumAddress;
              obj.gasUsed = gasUsed;
              obj.tokenName = tokenName;
              obj.tokenSymbol = tokenSymbol;
              obj.adjBalance = newbal;
              obj.balance = parseInt(balance);
              obj.dec = parseInt(dec);
              obj.imgURL = `https://raw.githubusercontent.com/trustwallet/assets/master/blockchains/ethereum/assets/${checkSumAddress}/logo.png`;

              tokenInfo[contractAddress] = obj;
              // console.table(tokenName, checkSumAddress, balance);
            } else {
              //console.log(tokenName, " no balance");
              omit[contractAddress] = true;
            }
          }
        }

        let addresses = Object.keys(tokenInfo).join(",");
        let url = `https://api.coingecko.com/api/v3/simple/token_price/ethereum?contract_addresses=${addresses}&vs_currencies=usd,eth`;
        let priceData = await fetch(url).then((response) => response.json());

        for (const [key, value] of Object.entries(tokenInfo)) {
          if (!priceData.hasOwnProperty(key)) {
            //console.log("deleting ", key);
            delete tokenInfo[key];
          } else {
            let prices = priceData[key];
            tokenInfo[key].prices = prices;
            let worth = prices.usd * tokenInfo[key].adjBalance;
            console.log(tokenInfo[key].tokenName, worth);
          }
        }

        this.setData(accounts[0], tokenInfo);
        //console.log(tokenInfo);
        // Object.keys(tokenInfo).map((cont) => {
        //   let token = tokenInfo[cont];

        //   try {
        //     let value = token.adjBalance * token.prices.usd;
        //     let url = `https://raw.githubusercontent.com/trustwallet/assets/master/blockchains/ethereum/assets/${token.checkSumAddress}/logo.png`;
        //     let img = document.createElement("IMG");

        //     img.src = url;
        //     img.width = value * 0.2;
        //     tokendisplay.append(img);
        //   } catch (e) {
        //     console.log("No image found");
        //   }
        // });
      };

      const handleNewAccounts = async (newAccounts) => {
        accounts = newAccounts;
        updateButtons();
      };

      const onClickConnect = async () => {
        try {
          const newAccounts = await ethereum.request({
            method: "eth_requestAccounts",
          });
          handleNewAccounts(newAccounts);
        } catch (error) {
          console.error(error);
        }
      };

      const updateButtons = () => {
        if (isMetaMaskConnected()) {
          onboardButton.innerText = "Connected";
          this.address = accounts[0];
          //getAccountsResult.innerHTML = accounts[0];
          onboardButton.disabled = true;
          let url = `https://api.etherscan.io/api?module=account&action=tokentx&address=${accounts[0]}&startblock=0&endblock=999999999&sort=asc&apikey=JB3WCP3RZRBR4Y2QYZ7P3S1A414AP39UEE`;
          fetch(url)
            .then((response) => response.json())
            .then((data) => parseTokens(data.result));
          this.parsingWallet();
        } else {
          onboardButton.innerText = "Connect";
          onboardButton.onclick = onClickConnect;
          onboardButton.disabled = false;
        }
      };

      // Change button text if MM installed or not
      const MetamaskClientCheck = async () => {
        //Now we check to see if Metmask is installed
        if (!isMetaMaskInstalled()) {
          //If it isn't installed we ask the user to click to install it
          onboardButton.innerText = "Click here to install MetaMask!";
          //When the button is clicked we call th is function
          onboardButton.onclick = onClickInstall;
          //The button is now disabled
          onboardButton.disabled = false;
        } else {
          //If MetaMask is installed we ask the user to connect to their wallet
          onboardButton.innerText = "Connect";
          //When the button is clicked we call this function to connect the users MetaMask Wallet
          onboardButton.onclick = onClickConnect;
          //The button is now disabled
          onboardButton.disabled = false;
        }
      };
      MetamaskClientCheck();

      if (isMetaMaskInstalled()) {
        ethereum.autoRefreshOnNetworkChange = false;
        try {
          const newAccounts = await ethereum.request({
            method: "eth_accounts",
          });
          handleNewAccounts(newAccounts);
        } catch (err) {
          console.error("Error on init when getting accounts", err);
        }
      }
    },
  },
};
</script>

<style scoped>
</style>