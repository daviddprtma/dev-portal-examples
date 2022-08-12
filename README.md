# Dev Portal Examples
Next, let’s install the required npm packages for the web application. To install the required npm packages, run the following command on your terminal.

npm install

Now that we have installed all the required packages, it’s time to start up a local environment of the web application. Run the following command on your terminal.

npm start

Once this is done, you can head to http://localhost:3000 to view your web application.

This is a simple web application that allows us to call the setHello and getHello transitions in our Hello World smart contract. We can connect our ZilPay wallet to the web application by clicking the “Connect Zilpay” button. 

Once your ZilPay wallet is connected, we can proceed to submit our contract address. This contract address is the one that is generated when we deployed our Hello World smart contract onto the Zilliqa testnet. This tells the web application which contract to make the transition calls to.

Copy and paste the address of the Hello World contract which you deployed in Quest 3 into the “New Address” field and press “Submit”. Check that there is no whitespace in front of the address, having a whitespace there will result in an error later on.



Once we have submitted the contract address, it will be shown at the top of the application.

Now, we can proceed to invoke the transitions to the contract. Let’s start by invoking getHello by clicking the “Get Hello” button. This will retrieve the current welcome message stored by our contract - which should be “Hello World” from the previous quest. Note that It might take up to a minute before the message appears.



Viola! We can see that the current welcome message is “Hello World”.

Note: as a transaction is only confirmed once a block is processed, it may take some time for the status to be confirmed and returned back. 

You can queue multiple transactions in the same block by increasing the nonce number by one for each transaction, but you have to be mindful each transaction will be processed in order. Imagine if we queued up two setHello transactions for our wallet 0x123456, the first transaction with nonce X is Alice, and the second transaction with nonce X+1 is Bob. 

When the chain processes all of the transactions for a block, X is processed first and Alice is written to state, followed immediately by X+1, writing Bob to the state. Users will never see the state of Alice as it’s immediately overwritten in the same block. This is why you have to be mindful of the order of transactions that will be processed.

Now back to the web app. Similar to what we did on Neo Savant IDE, we can change the welcome message of the smart contract using the setHello transition.



Type in your name and click on “Set Hello” to change the welcome message!

In the next few steps, we will go through the code that allows the web application to execute the various actions.
Step 5: Connecting to ZilPay Wallet
To connect your ZilPay wallet, the web application uses the ZilPay global API. Find out more about how the global API works from the ZilPay API documentation.

Let’s start by looking at “App.js” in the “hello-world/src” directory. We will go step-by-step to take you through how the various functions of the web application are implemented.
The code below contains the code for the button component that is labelled “Connect ZilPay”, used to connect your ZilPay wallet to the web application.

{!localStorage.getItem("zilpay_connect") && Connect Zilpay}


The first expression !localStorage.getItem("zilpay_connect") checks if you have already connected ZilPay to the web application. If you have not, the “Connect Zilpay” button will then be displayed for you to connect your ZilPay wallet. When the “Connect Zilpay” button is clicked, it executes a function this.connectZilpay.

async connectZilpay(){
  try {
   await window.zilPay.wallet.connect();
   if(window.zilPay.wallet.isConnect){
    localStorage.setItem("zilpay_connect", true);
    window.location.reload(false);
   } else {
   alert("Zilpay connection failed, try again...")
  }
  } catch (error) {}
 }


Above shows the full code for the connectZilpay function. Let’s break it down.

await window.zilPay.wallet.connect();


Firstly, the program tries to asynchronously establish a connection with a ZilPay wallet.

if(window.zilPay.wallet.isConnect){
    localStorage.setItem("zilpay_connect", true);
    window.location.reload(false);
   } else {
   alert("Zilpay connection failed, try again...")


If the connection is established successfully, it tells the application that it has successfully connected to a ZilPay wallet by setting the zilpay_connect value in your local storage to true. 

If the connection is not established successfully, it alerts the user that connection to the ZilPay wallet has failed, prompting the user to try again.
Step 6: Updating the Contract Address
Here are the components that allow the user to tell the web application which contract to invoke the transitions to.

<div>
 {" "}
 {`Current Contract Address : ${localStorage.getItem(
  "contract_address"
 )}`}{" "}
</div>
<h3>Update Contract Address</h3>
<form onSubmit={this.handleSubmit}>
 <label>
  New Address <br />
  <input
   type="text"
   onChange={this.handleAddressChange}
   size="70"
   placeholder="Format: 0x47d9CEea9a2DA23dc6b2D96A16F7Fbf884580665"
  />
 </label>
 <br />
 <input type="submit" value="Submit" />
 <hr></hr>
</form>

Let’s break it down!

<div>
 {" "}
 {`Current Contract Address : ${localStorage.getItem(
  "contract_address"
 )}`}{" "}
</div>

This div component retrieves the contract address from local storage and displays it onto the web application. If there is no contract address stored, it will be displayed as “null”.

<h3>Update Contract Address</h3>

This is just text to tell the user to update the contract address in the form below.

<form onSubmit={this.handleSubmit}>
 <label>
  New Address <br />
  <input
   type="text"
   onChange={this.handleAddressChange}
   size="70"
   placeholder="Format: 0x47d9CEea9a2DA23dc6b2D96A16F7Fbf884580665"
  />
 </label>
 <br />
 <input type="submit" value="Submit" />
 <hr></hr>
</form>

This form component is responsible for storing the contract address inputted by the user into local state. When the input component is changed, it calls the function this.handleAddressChange, which is responsible for storing the input value as local state in the web application. 

It then executes the this.handleSubmit function when the form is submitted. Here is the implementation for this.handleSubmit.

handleSubmit() {
 localStorage.setItem("contract_address", this.state.contractAddress);
}

Once the user submits the form, the above function is executed and the contract address obtained from the local state of the web application is stored into local storage. The contract address will be retrieved from local storage when invoking the setHello and getHello transitions, which will be explained further in the next steps.
Step 7: getHello Transition (Part 1 - getHello Function)
Up till this point, we have covered how the web application connects to ZilPay and how it stores the contract address that is submitted by the user. Now, we will look at how the web application invokes transitions to the Hello World smart contract. Let’s start by looking at the components.

<label>Get Hello</label>
<br />
<button onClick={this.getHello}>Get Hello</button>
<br />
<br />
<div> {`Current Welcome Msg : ${this.state.welcomeMsg}`} </div>

To invoke the getHello transition, we are merely rendering a button component that calls a function this.getHello when clicked, and retrieving the welcome message from the local state to be displayed onto the web application.. Let’s look at the getHello function.

async getHello() {
 if (window.zilPay.wallet.isEnable) {
  this.getWelcomeMsg();
 } else {
  const isConnect = await window.zilPay.wallet.connect();
  if (isConnect) {
   this.getWelcomeMsg();
  } else {
   alert("Not able to call setHello as transaction is rejected");
  }
 }
}

Let’s break it down.

if (window.zilPay.wallet.isEnable) {
  this.getWelcomeMsg();
 }

The function first checks if there is a ZilPay wallet connected to the web application by using window.zilPay.wallet.isEnable, which returns true or false depending on whether a ZilPay wallet is connected. If there is a ZilPay wallet connected to the web application, the welcome message is retrieved with this.getWelcomeMsg.

else {
   const isConnect = await window.zilPay.wallet.connect();
   if (isConnect) {
    this.getWelcomeMsg();
   } else {
    alert("Not able to call getHello as transaction is rejected");
   }
  }
 }

If there is no ZilPay wallet connected, it will try to establish a connection with a ZilPay wallet. If the connection is successful, it runs this.getWelcomeMsg to get the welcome message. Else, the user is alerted that the getHello transition call was unsuccessful.
Step 8: getHello Transition (Part 2 - getWelcomeMsg Function)
Now, let’s look at the code for getWelcomeMsg.

async getWelcomeMsg() {
 const zilliqa = window.zilPay;
 let contractAddress = localStorage.getItem("contract_address");
 const CHAIN_ID = 333;
 const MSG_VERSION = 1;
 const VERSION = bytes.pack(CHAIN_ID, MSG_VERSION);
 const myGasPrice = units.toQa("2000", units.Units.Li); // Gas Price that will be used by all transactions
 contractAddress = contractAddress.substring(2);
 const ftAddr = toBech32Address(contractAddress);
 try {
  const contract = zilliqa.contracts.at(ftAddr);
  const callTx = await contract.call("getHello", [], {
   // amount, gasPrice and gasLimit must be explicitly provided
   version: VERSION,
   amount: new BN(0),
   gasPrice: myGasPrice,
   gasLimit: Long.fromNumber(10000),
  });
  console.log(JSON.stringify(callTx.TranID));
  this.eventLogSubscription();
 } catch (err) {
  console.log(err);
 }
}

Let’s break down the code.

First, a few variables which are used in the contract call are declared. 

const zilliqa = window.zilPay;
let contractAddress = localStorage.getItem("contract_address");

window.zilPay is mapped to zilliqa - this will be used to execute the contract call later in the function.

contractAddress is retrieved from the local storage - recall that the contract address was stored into local storage in Step 5.

const CHAIN_ID = 333;
const MSG_VERSION = 1;
const VERSION = bytes.pack(CHAIN_ID, MSG_VERSION);

The chain ID of the Zilliqa testnet and the msg version are then packed into bytes with the bytes.pack function provided by zilliqa-js and mapped to the variable VERSION. This will be used later to tell the program which network to execute the contract call to.

const myGasPrice = units.toQa("2000", units.Units.Li);

The gas price that will be used in the contract call is defined here. units.toQa function takes in a string of numbers and a unit to be converted into QA. Here, 2000 LIs are converted into QA - recall that 1 LI is 106 QAs.

contractAddress = contractAddress.substring(2);
const ftAddr = toBech32Address(contractAddress);

Here, the contract address is converted from a Base16 Address to a Bech32 Address, which will be used for the contract call.

Now that we have declared the variables required for the contract call, it’s time to make the contract call.

try {
   const contract = zilliqa.contracts.at(ftAddr);
   const callTx = await contract.call("getHello", [], {
    version: VERSION,
    amount: new BN(0),
    gasPrice: myGasPrice,
    gasLimit: Long.fromNumber(10000),
   });
   console.log(JSON.stringify(callTx.TranID));
   this.eventLogSubscription();
  } catch (err) {
   console.log(err);
  }

Let’s break it down.

const contract = zilliqa.contracts.at(ftAddr);

The contract is first constructed with the contracts.at method provided by ZilPay, specifying the Bech32 Address of the user who makes the contract call.

const callTx = await contract.call("getHello", [], {
    version: VERSION,
    amount: new BN(0),
    gasPrice: myGasPrice,
    gasLimit: Long.fromNumber(10000),
   });

The contract is then called with the contract.call method, which takes in the parameters of the transaction as arguments to execute the contract call. Refer to the ZilPay documentation for more details.

The first argument ”getHello” specifies which transition to invoke in the contract call.

The second argument specifies the initial parameters to the transition call as an array of objects. For the getHello transition, there are no initial parameters to specify. Hence, an empty array is added.

The third argument specifies the transaction parameters of the contract call. Here, we specify the version, amount, gas price, and gas limit of the contract call that we will be invoking.

console.log(JSON.stringify(callTx.TranID));
this.eventLogSubscription();

Once the contract call has been executed, the transaction ID of the contract call is logged onto the console and the this.eventLogSubscription function is called. 

this.eventLogSubscription listens for the event emitted once the contract call has been successfully made and retrieves the welcome message to be displayed on the web application.
Step 9: setHello Transition
Now that we have understood how the getHello transition is implemented, let’s take a look at the setHello transition. Its implementation is largely similar to that of getHello, with a few minor differences. Let’s start by looking at the components.

<label>Set Hello</label>
<br />
<input type="text" onChange={this.handleHelloChange} size="30" />
<br />
<button onClick={this.setHello}>Set Hello</button>

To execute the setHello transition call, there is an input field for the user to key in a new welcome message and a button which invokes the transition call. As the user keys in the new welcome message into the input field, the this.handleHelloChange function is executed.

handleHelloChange(event) {
 this.setState({ setHelloValue: event.target.value });
}

The handleHelloChange function merely takes the new welcome message and stores it in local state to be used in the contract call. When the “Set Hello” button is clicked, the this.setHello function is executed.

async setHello() {
 if (window.zilPay.wallet.isEnable) {
  this.updateWelcomeMsg();
 } else {
  const isConnect = await window.zilPay.wallet.connect();
  if (isConnect) {
   this.updateWelcomeMsg();
  } else {
   alert("Not able to call setHello as transaction is rejected");
  }
 }
}

Similar to the getHello function, it checks if there is a ZilPay wallet connected to the web application. If there is, the this.updateWelcomeMsg function is executed. If not, it attempts to establish a connection with a ZilPay wallet. Below shows the code for updateWelcomeMsg.

async updateWelcomeMsg() {
 const zilliqa = window.zilPay;
 let setHelloValue = this.state.setHelloValue;
 let contractAddress = localStorage.getItem("contract_address");
 const CHAIN_ID = 333;
 const MSG_VERSION = 1;
 const VERSION = bytes.pack(CHAIN_ID, MSG_VERSION);
 const myGasPrice = units.toQa("2000", units.Units.Li); // Gas Price that will be used by all transactions
 contractAddress = contractAddress.substring(2);
 const ftAddr = toBech32Address(contractAddress);
 try {
  const contract = zilliqa.contracts.at(ftAddr);
  const callTx = await contract.call(
   "setHello",
   [
    {
     vname: "msg",
     type: "String",
     value: setHelloValue,
    },
   ],
   {
    // amount, gasPrice and gasLimit must be explicitly provided
    version: VERSION,
    amount: new BN(0),
    gasPrice: myGasPrice,
    gasLimit: Long.fromNumber(10000),
   }
  );
 } catch (err) {
  console.log(err);
 }
}

The code here is largely similar to that of getWelcomeMsg for the getHello contract call. One notable difference is that the setHello contract call requires an initial parameter msg, which is the new welcome message that will be stored on the deployed contract.

[
 {
  vname: "msg",
  type: "String",
  value: setHelloValue,
 },
]

The initial parameter is added into the second argument of the contract.call method as an object in the array - with the name of the initial parameter as vname, the data type of the parameter as type, and the value of the initial parameter as value. For contract calls with more than one initial parameter, multiple objects can be added into the array.

Those with keen eyes may realise that there is no this.eventLogSubscription function in updateWelcomeMsg. This function is not needed for the setHello transition as we are not retrieving any data from the event emitted by the contract call.
Step 10: Viewing Transition Calls with Transaction ID
Before we wrap up this quest, let’s explore one last concept - Transaction IDs. Put simply, a Transaction ID is a unique identifier for every transaction that happens on the Zilliqa network.

console.log("Transaction ID: " + callTx.ID);


The contract.call method returns an object with information about the contract call made to the blockchain. We can retrieve the transaction ID of the contract call by adding the above line of code below the contract.call method. This will display the transaction ID in the terminal when the setHello transition call is made. Refer to the image in ‘Working code’ if you are unsure.

First, let’s refresh the web application to deploy the changes. Next, let’s invoke the “Set Hello” transition again. This time, let’s set the message as “hello your-name” this time. Once again, confirm your transaction with ZilPay.



To view the transaction ID of the transition call, let’s open up the browser console. Check out this guide on how to open up your browser console for your operating system and browser. 

You should see your transaction ID being displayed in the console. Let’s head to Devex to view the transaction that we have just made. To search for the transaction with the transaction ID retrieved from callTx, we have to add in “0x” at the front, e.g. 0xc978d38d7eee3d9e7a233baa43d1f725471d544ad0101cedce751fe50ca56f09.

After you’ve searched for your transaction, you will be able to see information regarding your transaction such as the address that the transaction was sent from, the address it is sent to, the parameters of the transaction and so on.

That’s all for the Hello World web application! 
