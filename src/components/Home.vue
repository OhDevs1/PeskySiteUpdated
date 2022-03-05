<template>
<div>
    <b-container class="mintHub">
        <b-row>
            <b-col cols="12">
                <h2 class="Header">Mint A Panda</h2>
                <br>
                <img v-if="!isMobile()" src="../assets/pandas.gif" alt="">
                <img v-if="isMobile()" class="gifImage" src="../assets/pandas.gif" alt="">
                <br><br>
                <h4 class="Header2">{{totalSupply}} / 1,111 Minted!</h4>
                <br>
                <h4 class="Header2">0.15 ETH Each</h4>
                <br>
                <div v-if="!loadingTransaction">
                    <input class="inputArea" type="Number" min="1" v-model="mintAmount">
                    <button @click="mintShark()" class="bbrc-button mint">Mint</button>
                </div>
                <br>
                <b-icon v-if="loadingTransaction" icon="arrow-clockwise" animation="spin-pulse" font-scale="4"></b-icon>
            </b-col>
        </b-row>
        <br><br><br><br><br>
    </b-container>
</div>
</template>

<script>
import Web3 from 'web3';
import contractInfo from '../assets/contract.json'

export default {
  name: "Home",
  data: function () {
    return {
      walletAddress: '',
      walletBalance: '',

      sharkContract: '',
      contractAbi: contractInfo.abi,
      contractAddress: contractInfo.contractAddress,

      mintPrice: 0.15,
      mintAmount: 1,
      presaleOpen: false,
      mainsaleOpen: false,
      totalSupply: '',

      loadingTransaction: false,

      web3:'',

      tempErr: '',
    }
  },
  
  mounted()
  {
      this.LoadEthWallet();
  },
  methods: 
  {
    isMobile() {
      if(/Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)) {
        return true
      } else {
        return false
      }
    },
    mintShark()
    {
      this.loadingTransaction = true;
      if(this.presaleOpen)
		  {
        const weiValue = this.web3.utils.toWei((this.mintAmount * 0.15).toString(), 'ether');

        this.sharkContract.methods.presale(this.mintAmount).send({from: this.walletAddress, value: weiValue })
        .on('confirmation', function(confirmationNumber, receipt)
        {
          
        })
        .then(res=>{
          alert("You minted Strange Shark #" + res.events.Transfer.returnValues[2])
          this.LoadContract();
          this.loadingTransaction = false;
        })
        .catch(err=>{
          this.LoadContract();
          this.loadingTransaction = false;

        })
      }
      else if(this.mainsaleOpen)
      {

        const weiValue = this.web3.utils.toWei((this.mintAmount * 0.15).toString(), 'ether');

        this.sharkContract.methods.mainsale(this.mintAmount).send({from: this.walletAddress, value: weiValue })
        .on('confirmation', function(confirmationNumber, receipt)
        {
          
        })
        .then(res=>{

          alert("You minted Strange Shark #" + res.events.Transfer.returnValues[2])
          this.LoadContract();
          this.loadingTransaction = false;
        })
        .catch(err=>{
          this.LoadContract();
          this.loadingTransaction = false;

        })
      }
      else
      {
        alert("Minting is Paused!");
      }
    },
    LoadContract: function()
        {
            this.sharkContract = new this.web3.eth.Contract(this.contractAbi, this.contractAddress);

            this.sharkContract.methods.sharksPresaleOpen().call({from: this.walletAddress})
            .then(res=>{
                this.presaleOpen = res;
            })
            this.sharkContract.methods.sharksMainsaleOpen().call({from: this.walletAddress})
            .then(res=>{
                this.mainsaleOpen = res;
            })
            this.sharkContract.methods.totalSupply().call({from: this.walletAddress})
            .then(res=>{
                this.totalSupply = res;
            })
        },
        LoadEthWallet: function()
        {
            if (typeof window.ethereum !== 'undefined') 
            {
                this.web3 = new Web3(window.ethereum || new Web3.providers.HttpProvider("https://mainnet.infura.io/v3/1xVMLkEDl6htSmYN7YeFXGvIe07") || Web3.givenProvider);
                this.web3.eth.net.getNetworkType()
                .then(network=>{
                    if(network == "main")
                    {   
                        this.web3.eth.requestAccounts()
                        .then(acc=>{
                        this.walletAddress = acc[0];
                        this.web3.eth.getBalance(this.walletAddress)
                        .then(balance=>{
                            this.walletBalance = this.web3.utils.fromWei(balance, "ether");  
                            this.LoadContract();  
                        });
                    })
                    .catch(err=>{
                        console.log(err);
                    })            
                }
                else{
                    alert("Switch to mainnet!")
                }          
            })
            }
        }  
  }
}
</script>

<style scoped>
.gifImage
{
  width: 100%;
}
.inputArea
{
  width: 10%;
}
.mintHub
{
    margin-top: 2%;
}
.Header
{
    color: #00dc7a;
    font-weight: bolder;
    font-size: 5vh;
}
.Header2
{
    color: #00dc7a;
    font-weight: bolder;
    font-size: 3vh;
}
.mint {
    background-color: black;
    color: #00dc7a;
    font-weight: 600;
}
.bbrc-button {
    background-color: white;
    border-radius: 8px;
    color: #00dc7a;
    border: 0;
    font-weight: 700;
    font-size: 20px;
    padding: 1%;
    margin: auto auto;
    transition: .3s;
}
</style>