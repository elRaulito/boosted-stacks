<script setup>
import { ref, onMounted } from 'vue';
import { showConnect, UserSession, openContractCall } from "@stacks/connect";
import { Cl, PostConditionMode,fetchCallReadOnlyFunction } from "@stacks/transactions";

// Initialize the session
const userSession = new UserSession();

// States for the Lend form
const isConnected = ref(false);
const positionData = ref(null);
const borrowData= ref(null);
const depositAmount = ref(0); // Amount to deposit (0 to 1 sBTC)
const depositAPY = ref(5); // APR for the deposit
const depositDays = ref(30); // Days for the deposit (30, 60, 120, 300)
const userAddress =ref(""); 

const borrowAmount = ref(0); // Amount to deposit (0 to 1 sBTC)
const borrowDays=ref(30)

const withdrawAmount = ref(0); // Amount to deposit (0 to 1 sBTC)

// Data for the bar chart (liquidity for sBTC across different APYs)
const apyData = [
  { apy: 4, liquidity: 0},
  { apy: 5, liquidity: 20 },
  { apy: 6, liquidity: 40 },
  { apy: 7, liquidity: 50 },
  { apy: 8, liquidity: 20 },
  { apy: 9, liquidity: 15 },
  { apy: 10, liquidity: 5 },
];

// Initialize the chart
const chartRef = ref(null);

// States for text visibility
const lendTextVisible = ref(false);
const borrowTextVisible = ref(false);

// Handle Connect and Disconnect
const handleConnect = async () => {
  await showConnect({
    appDetails: {
      name: "Boosted Stacks",
      icon: '/logo.svg',
    },
    onFinish: () => {
      isConnected.value = true;
      
      const userData = userSession.loadUserData();
      console.log(userData.profile.stxAddress)
      userAddress.value = userData.profile.stxAddress.mainnet;  // Get the user's address
      console.log(userData.profile);
      handleGetData()
      handleGetDataBorrowers()
    },
  });
};

const handleDisconnect = () => {
  userSession.signUserOut();
  isConnected.value = false;
  userAddress.value = "";  // Clear the address when disconnected
};

// Handle Lend and Borrow actions
const handleLendClick = () => {
  lendTextVisible.value = !lendTextVisible.value; // Toggle text visibility
};

const handleGetData = async () => {
  // get the pool id from the pool initial data
  const poolIdResult = await fetchCallReadOnlyFunction({
    contractAddress: "SP3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N6R5192Q",
    contractName: "finalboosted",
    functionName: "get-lenders",
    functionArgs: [
      Cl.principal(userAddress.value)
    ],
    senderAddress: userAddress.value,
    network: "mainnet"
  });

  // Parse poolIdResult.value as a JSON object
  const data = poolIdResult.value;
  if(poolIdResult.type=="tuple"){

  console.log(data)
  // Extract the values from the object
  const amount = parseInt(data.amount.value);
  const expiration = parseInt(data.expiration.value);
  const uDailyprice = parseInt(data.uDailyprice.value);

  console.log("Amount:", amount);
  console.log("Expiration:", expiration);
  console.log("Daily Price:", uDailyprice);
  let arrayBorrowers= []

  // Loop through the borrowers array and extract the values for each borrower
  data.borrowers.value.forEach(borrower => {
    console.log("sono dentro al loop")
    const borrowerAddress = borrower.value.borrower.value;
    const borrowerAmount = parseInt(borrower.value.amount.value);
    const borrowerDays = parseInt(borrower.value.days.value);
    const borrowerDeadline = parseInt(borrower.value.deadline.value);
    arrayBorrowers.push({
      "borrower" : borrowerAddress,
      "amount": borrowerAmount,
      "days":borrowerDays,
      "deadline":borrowerDeadline
    })

    console.log(`Borrower Address: ${borrowerAddress}`);
    console.log(`Amount: ${borrowerAmount}`);
    console.log(`Days: ${borrowerDays}`);
    console.log(`Deadline: ${borrowerDeadline}`);
  });
  console.log(data)
  positionData.value = {
          amount: amount,
          expiration: expiration,
          dailyPrice: uDailyprice,
          borrowers: arrayBorrowers
        };
  console.log(positionData.value)
  console.log(positionData.value.borrowers.length)
  }
};




const handleGetDataBorrowers = async () => {
  // get the pool id from the pool initial data
  const poolIdResult = await fetchCallReadOnlyFunction({
    contractAddress: "SP3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N6R5192Q",
    contractName: "finalboosted",
    functionName: "get-borrowers",
    functionArgs: [
      Cl.principal(userAddress.value)
    ],
    senderAddress: userAddress.value,
    network: "mainnet"
  });

  // Parse poolIdResult.value as a JSON object
  const data = poolIdResult.value;
  let arrayBorrowers= []

  // Loop through the borrowers array and extract the values for each borrower
  data.borrowers.value.forEach(borrower => {
    console.log("sono dentro al loop")
    const borrowerAddress = borrower.value.borrower.value;
    const borrowerAmount = parseInt(borrower.value.amount.value);
    const borrowerDays = parseInt(borrower.value.days.value);
    const borrowerDeadline = parseInt(borrower.value.deadline.value);
    const owner=borrower.value.owner.value
    arrayBorrowers.push({
      "borrower" : borrowerAddress,
      "amount": borrowerAmount,
      "days":borrowerDays,
      "owner":owner,
      "deadline":borrowerDeadline
    })

    console.log(`Borrower Address: ${borrowerAddress}`);
    console.log(`Amount: ${borrowerAmount}`);
    console.log(`Days: ${borrowerDays}`);
    console.log(`Deadline: ${borrowerDeadline}`);
  });
  console.log(data)
  borrowData.value = {
          borrowers: arrayBorrowers
        };
  console.log(borrowData.value)
  console.log(borrowData.value.borrowers.length)
};


// Handle Deposit Form Submission
const handleDepositSubmit = () => {
  console.log(depositAmount.value)
  openContractCall({
    contractAddress: "SP3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N6R5192Q",
    contractName: "finalboosted",
    functionName: "deposit_final", // Assuming a function for deposit
    functionArgs: [
      Cl.uint(depositAmount.value * 1e8), // Convert to smallest unit
      Cl.uint(depositDays.value),
      Cl.uint(depositAPY.value * 39000000), // Convert to percentage form (e.g., 5% = 500)
    ],
    postConditionMode: PostConditionMode.Allow,
    network: "mainnet",
  });
};



// Handle Deposit Form Submission
const handleWithdraw = () => {
  console.log(withdrawAmount.value)
  openContractCall({
    contractAddress: "SP3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N6R5192Q",
    contractName: "finalboosted",
    functionName: "withdraw_final", // Assuming a function for deposit
    functionArgs: [
      Cl.uint(withdrawAmount.value * 1e8), // Convert to smallest unit
    ],
    postConditionMode: PostConditionMode.Allow,
    network: "mainnet",
  });
};



// Handle Deposit Form Submission
const handleBorrowSubmit = () => {
  openContractCall({
    contractAddress: "SP3NBRSFKX28FQ2ZJ1MAKX58HKHSDGNV5N6R5192Q",
    contractName: "finalboosted",
    functionName: "borrow", // Assuming a function for deposit
    functionArgs: [
      Cl.uint(borrowAmount.value * 1e8), // Convert to smallest unit
      Cl.uint(borrowDays.value),
    ],
    postConditionMode: PostConditionMode.Allow,
    network: "mainnet",
  });
};



const handleBorrowClick = () => {
  borrowTextVisible.value = !borrowTextVisible.value; // Toggle text visibility
};

const renderChart = () => {
  const ctx = chartRef.value.getContext('2d');
  const barWidth = 40; // Width of each bar
  const spaceBetweenBars = 15; // Space between bars

  // Get the maximum liquidity value to scale the bars
  const maxLiquidity = Math.max(...apyData.map(data => data.liquidity));

  // Create a gradient for the bars (using linear gradient)
  const gradient = ctx.createLinearGradient(0, 0, 0, 150);
  gradient.addColorStop(0, 'rgba(255, 102, 0, 0.84)'); // Start color (orange)
  gradient.addColorStop(1, 'rgba(245, 203, 126, 0.46)'); // End color (light orange)

  // Loop through the apyData to draw bars
  apyData.forEach((data, index) => {
    const barHeight = (data.liquidity / maxLiquidity) * 150; // Scale bar height
    const xPosition = index * (barWidth + spaceBetweenBars); // X position for each bar
    const yPosition = 200 - barHeight; // Y position for each bar, inverted to draw from bottom

    // Draw bars with the gradient color
    ctx.fillStyle = gradient;
    ctx.fillRect(xPosition, yPosition, barWidth, barHeight); // Draw bar

    // Add labels (APY) under each bar
    ctx.fillStyle = '#000'; // Label color (black)
    ctx.font = '12px Arial';
    ctx.fillText(`${data.apy}%`, xPosition + barWidth / 4, 215); // Position of the label
  });

  // Draw X and Y axes
  ctx.strokeStyle = '#000'; // Axis color (black)
  ctx.lineWidth = 2;

  // Draw X-axis
  ctx.beginPath();
  ctx.moveTo(30, 200); // Starting point of X-axis
  ctx.lineTo(380, 200); // Ending point of X-axis
  ctx.stroke();

  // Draw Y-axis
  ctx.beginPath();
  ctx.moveTo(30, 0); // Starting point of Y-axis
  ctx.lineTo(30, 200); // Ending point of Y-axis
  ctx.stroke();

  // Add values to the Y axis (for liquidity scale)
  for (let i = 0; i <= 10; i++) {
    const yValue = (i / 10) * maxLiquidity; // Calculate Y values
    ctx.fillStyle = '#000';
    ctx.font = '10px Arial';
    ctx.fillText(`${yValue} sBTC`, 10, 200 - (i * 20)); // Display values on Y-axis
  }

  // Add values to the X axis (for APY)
  apyData.forEach((data, index) => {
    const xPosition = index * (barWidth + spaceBetweenBars);
    ctx.fillStyle = '#000';
    ctx.font = '12px Arial';
    ctx.fillText(`${data.apy}%`, xPosition + barWidth / 4, 215); // Position of the label on X-axis
  });
};


onMounted(() => {
  renderChart();
});


const formatDate = (timestamp) => {
const date = new Date(timestamp * 1000);  // Convert Unix timestamp to milliseconds
      return date.toLocaleString();  // Convert to a human-resBTCble string
};


  
</script>

<template>
  <div class="dex-container" @mousemove="moveGradient">
    <!-- Font Preconnect and Link -->
    <header>
      <link rel="preconnect" href="https://fonts.googleapis.com">
      <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
      <link href="https://fonts.googleapis.com/css2?family=Pixelify+Sans:wght@400..700&display=swap" rel="stylesheet">
      
      <div class="header-content">
        <h1>Boosted Stacks</h1>
        <p class="subtitle">Boost your sBTC yield</p>
      </div>
    </header>

    <!-- Connect / Disconnect Button -->
    <button class="connect-btn" @click="isConnected ? handleDisconnect() : handleConnect()">
      {{ isConnected ? 'Disconnect Wallet' : 'Connect Wallet' }}
    </button>

    <!-- Main Content Section -->
    <section class="trade-section" :class="{ 'blurred': !isConnected }">
      <!-- Left Side: Lend / Borrow Section -->
      <div class="left-side">
        <div class="action-card">
          <h2>Deposit sBTC</h2>
          <p v-if="lendTextVisible">Here you can lend your sBTC at the best APY rates. <br>
          sBTC won't be spendable by borrowers, but they will get your sBTC rewards
          </p> <!-- Text before the button -->
          
          <!-- Deposit Amount Slider -->
          <label for="depositAmount" v-if="lendTextVisible">Deposit Amount (sBTC):</label>
          <input type="range" id="depositAmount" v-model="depositAmount" min="0" max="0.01" step="0.0001" v-if="lendTextVisible"/>
          <p v-if="lendTextVisible">{{ depositAmount }} sBTC</p>

          <!-- APR Slider -->
          <label for="depositAPY" v-if="lendTextVisible">APR (%):</label>
          <input type="range" id="depositAPY" v-model="depositAPY" min="4" max="20" step="0.1" v-if="lendTextVisible"/>
          <p v-if="lendTextVisible">{{ depositAPY }}%</p>

          <!-- Days Selection -->
          <label for="depositDays" v-if="lendTextVisible">Deposit Days:</label>
          <div v-if="lendTextVisible">
            <div class="button-grid">
              <button @click="depositDays = 30" :class="{ active: depositDays === 30 }">30</button>
              <button @click="depositDays = 60" :class="{ active: depositDays === 60 }">60</button>
              <button @click="depositDays = 120" :class="{ active: depositDays === 120 }">120</button>
              <button @click="depositDays = 300" :class="{ active: depositDays === 300 }">300</button>
            </div>
          </div>
          <p v-if="lendTextVisible">{{ depositDays }} Days</p>
          <button class="action-btn" @click="handleDepositSubmit" v-if="lendTextVisible">Deposit sBTC</button>
          <button class="action-btn" @click="handleLendClick">Deposit</button>
        </div>
        <div class="action-card">
          <h2>Borrow sBTC yield</h2>
          <p v-if="borrowTextVisible">Here you can borrow sBTC with minimal fees.</p> <!-- Text before the button -->

          <!-- Deposit Amount Slider -->
          <label for="borrowAmount" v-if="borrowTextVisible">Borrow yield Amount (sBTC):</label>
          <input type="range" id="borrowAmount" v-model="borrowAmount" min="0" max="0.01" step="0.0001" v-if="borrowTextVisible"/>
          <p v-if="borrowTextVisible">{{ borrowAmount }} sBTC</p>

          <!-- Days Selection -->

          <label for="borrowDays" v-if="borrowTextVisible">Borrow Days:</label>
          <div v-if="borrowTextVisible">
            <div class="button-grid">
              <button @click="borrowDays = 7" :class="{ active: borrowDays === 7 }">7</button>
              <button @click="borrowDays = 30" :class="{ active: borrowDays === 30 }">30</button>
              <button @click="borrowDays = 60" :class="{ active: borrowDays === 60 }">60</button>
              <button @click="borrowDays = 120" :class="{ active: borrowDays === 120 }">120</button>
            </div>
          </div>
          <p v-if="borrowTextVisible">{{ borrowDays }} Days</p>
          <button class="action-btn" @click="handleBorrowSubmit" v-if="borrowTextVisible">Borrow sBTC</button>
          <button class="action-btn" @click="handleBorrowClick">Borrow</button>
        </div>
      </div>

      <!-- Right Side: Market & Your Position Section -->
      <div class="right-side">
        <!-- Market Section Card -->
        <div class="trade-card">
          <h2>Market: Liquidity for sBTC</h2>
          <canvas ref="chartRef" width="400" height="200"></canvas>
        </div>

        <!-- Your Position Section -->
        <div class="trade-card">
          <h2>Your Position</h2>
        <p v-if="isConnected">User Address: <strong>{{ userAddress }}</strong></p>
  
  <div v-if="borrowData">
    <h4>Active Loans as borrower:</h4>
    <ul>
      <li v-for="(borrower, index) in borrowData.borrowers" :key="index">
        <p><strong>Lender Address:</strong> {{ borrower.owner }}</p>
        <p><strong>Amount:</strong> {{ borrower.amount }} sBTC</p>
        <p><strong>Days loan:</strong> {{ borrower.days }} days</p>
        <p><strong>Deadline:</strong> {{ formatDate(borrower.deadline) }}</p>
      </li>
    </ul>
  </div>



          <p v-else>Connect your wallet to see your position</p>

<div v-if="positionData">
  <h3>Deposited Amount: {{ positionData.amount }} sBTC</h3>
  <p>Expiration: {{ formatDate(positionData.expiration) }}</p>
  <p>APY %: {{ positionData.dailyPrice/39000000 }} (Paid in STX)</p>

 <label for="withdrawAmount" v-if="positionData">withdraw amount (sBTC):</label>
          <input type="range" id="withdrawAmount" v-model="withdrawAmount" min="0" max="0.01" step="0.0001" v-if="positionData"/>
          <p v-if="positionData">{{ withdrawAmount }} sBTC</p>
          <button v-if="positionData" class="action-btn" @click="handleWithdraw">Withdraw</button>


  <div v-if="positionData.borrowers.length">
    <h4>Active Loans:</h4>
    <ul>
      <li v-for="(borrower, index) in positionData.borrowers" :key="index">
        <p><strong>Borrower Address:</strong> {{ borrower.address }}</p>
        <p><strong>Amount:</strong> {{ borrower.amount }} sBTC</p>
        <p><strong>Days loan:</strong> {{ borrower.days }} days</p>
        <p><strong>Deadline:</strong> {{ formatDate(borrower.deadline) }}</p>
      </li>
    </ul>
  </div>
  <div v-else>
    <p>No active loans as lender</p>
  </div>
</div>

        </div>

      </div>

    </section>
  </div>
</template>

<style scoped>
/* Ensure the body and html take up the full height and width */
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  width: 100%;
  overflow: hidden; /* Hide scrollbars */
  background-image: url('.\assets\background.jpg');
  background-size: cover; /* Cover the whole page */
  background-position: center; /* Center the background */
  background-attachment: fixed; /* Make the background fixed when scrolling */
}

/* Full screen container with transparency */
.dex-container {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  padding: 20px;
  height: 100vh; /* Full height */
  width: 100vw;  /* Full width */
  font-family: 'Pixelify Sans', cursive; /* Retro font */
  position: relative;
  overflow: hidden;
  background-image: url('.\assets\background.jpg');
  background-size: cover; /* Cover the whole page */
  background-position: center; /* Center the background */
  background-attachment: fixed; /* Make the background fixed when scrolling */
}

.header-content {
  text-align: center;
  width: 100%;
}

h1 {
  color: #ff6600;  /* Orange color */
  font-size: 3rem;
  margin-bottom: 0;
}

.subtitle {
  color: rgb(255, 255, 255);
  font-size: 1.2rem;
  margin-top: 10px;
}

/* Connect Button */
.connect-btn {
  padding: 10px 20px;
  background-color: #ff6600;
  color: white;
  border: none;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 5px;
  position: absolute;
  top: 20px;
  right: 20px;
  width: auto;
}

.connect-btn:hover {
  background-color: #e55b00;
}

/* Trade section */
.trade-section {
  display: flex;
  justify-content: space-between;
  width: 100%;
  margin-top: 80px; /* Adjust to make space for the header */
}

.left-side {
  width: 30%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 20px;
}

.action-card {
  width: 100%;
  text-align: left;
  background-color: rgba(116, 116, 116, 0.55);/* Transparent background */
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.action-btn {
  padding: 10px;
  background-color:rgba(255, 104, 3, 0.64);
  color: white;
  border: none;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 5px;
  width: 100%;
  margin-top: 10px;
}

.action-btn:hover {
  background-color: #e55b00;
}

/* Right Side: Market & Your Position Section */
.right-side {
  width: 65%;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.trade-card {
  background-color: rgba(116, 116, 116, 0.64); /* Transparent background */
  border-radius: 10px;
  padding: 20px;
  width: 100%;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

h2 {
  color: rgb(255, 255, 255);
  font-size: 1.5rem;
  margin-bottom: 20px;
}

input[type="number"], select {
  width: 100%;
  padding: 10px;
  margin-top: 5px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

button {
  width: 100%;
  padding: 15px;
  border: none;
  background-color: #ff6600;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  border-radius: 5px;
}

button:disabled {
  background-color: #ddd;
}

button:hover {
  background-color: rgb(255, 173, 118);
}

/* Blur effect applied to all sections except the Market */
.trade-section.blurred {
  filter: blur(5px);
  pointer-events: none;
}

.button-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* 4 columns, each takes equal space */
  gap: 10px; /* space between the buttons */
  width: 100%;
}

.button-grid button {
  padding: 10px 20px;
  background-color:rgba(255, 102, 0, 0.48);
  color: white;
  border: none;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 50px;
  transition: background-color 0.3s ease;
}

.button-grid button.active {
  background-color: #e55b00; /* Active button color */
}

.button-grid button:hover {
  background-color: #e55b00;
}

</style>
