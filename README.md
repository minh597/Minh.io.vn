<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trò Chơi Clicker</title>
    <!-- Thư viện Tailwind CSS để tạo kiểu nhanh chóng -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font chữ Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #1a202c; /* Nền tối */
            color: #e2e8f0; /* Chữ sáng */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 1rem;
            box-sizing: border-box;
        }
        .game-container {
            background-color: #2d3748; /* Nền container */
            border-radius: 1.5rem;
            padding: 2rem;
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.2);
            width: 100%;
            max-width: 400px;
            text-align: center;
        }
        .click-button {
            width: 150px;
            height: 150px;
            border-radius: 50%;
            background: linear-gradient(135deg, #4299e1, #3182ce);
            border: none;
            box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2);
            transition: transform 0.1s ease-in-out, box-shadow 0.1s ease-in-out;
            margin-top: 1.5rem;
            margin-bottom: 2rem;
            font-size: 2.5rem;
            font-weight: bold;
            color: white;
            cursor: pointer;
            outline: none;
            display: inline-flex;
            justify-content: center;
            align-items: center;
            user-select: none;
        }
        .click-button:active {
            transform: scale(0.95);
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
        }
        .shop-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 100;
        }
        .shop-content {
            background-color: #2d3748;
            padding: 2rem;
            border-radius: 1.5rem;
            width: 90%;
            max-width: 500px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
            position: relative;
        }
        .shop-item {
            background-color: #4a5568;
            border-radius: 0.75rem;
            padding: 1rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 1rem;
        }
        .shop-item button {
            background-color: #48bb78;
            color: white;
            font-weight: bold;
            padding: 0.5rem 1rem;
            border-radius: 0.5rem;
            transition: background-color 0.2s;
            cursor: pointer;
            border: none;
        }
        .shop-item button:hover {
            background-color: #38a169;
        }
        .shop-item button:disabled {
            background-color: #a0aec0;
            cursor: not-allowed;
        }
        .message-box {
            background-color: #4a5568;
            color: white;
            padding: 1rem;
            border-radius: 0.75rem;
            margin-bottom: 1rem;
            display: none;
            opacity: 0;
            transition: opacity 0.3s ease-in-out;
        }
        .message-box.show {
            display: block;
            opacity: 1;
        }
    </style>
</head>
<body class="bg-gray-900 text-white">

<div class="game-container">
    <h1 class="text-3xl font-bold mb-2">Trò Chơi Clicker</h1>
    <p class="text-lg">Số tiền của bạn:</p>
    <div id="coin-display" class="text-5xl font-extrabold text-yellow-300 mt-2 mb-4">0</div>
    <button id="click-button" class="click-button">
        Bấm
    </button>
    <div class="flex justify-center space-x-4">
        <button id="shop-button" class="bg-indigo-500 hover:bg-indigo-600 text-white font-bold py-2 px-6 rounded-xl transition-colors">
            Cửa hàng
        </button>
    </div>
</div>

<!-- Shop Modal -->
<div id="shop-modal" class="shop-modal hidden">
    <div class="shop-content">
        <h2 class="text-2xl font-bold mb-4">Cửa Hàng</h2>
        <div id="message-box" class="message-box"></div>
        <div id="shop-items-container">
            <!-- Items will be dynamically added here -->
        </div>
        <button id="close-shop-button" class="mt-4 bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-6 rounded-xl transition-colors">
            Đóng
        </button>
    </div>
</div>

<script>
    // Game state variables
    let coins = 0;
    let clickPower = 1;
    let autoClickerRate = 0; // Coins per second

    // Game loop interval
    let autoClickerInterval;

    // References to HTML elements
    const coinDisplay = document.getElementById('coin-display');
    const clickButton = document.getElementById('click-button');
    const shopButton = document.getElementById('shop-button');
    const shopModal = document.getElementById('shop-modal');
    const closeShopButton = document.getElementById('close-shop-button');
    const shopItemsContainer = document.getElementById('shop-items-container');
    const messageBox = document.getElementById('message-box');

    // List of shop items with their properties
    const shopItems = [
        {
            id: 'auto-clicker-1',
            name: 'Máy bấm tự động',
            cost: 100,
            effect: { type: 'auto-click', value: 1 }
        },
        {
            id: 'click-power-1',
            name: 'Tăng sức mạnh bấm',
            cost: 50,
            effect: { type: 'click-power', value: 1 }
        },
        {
            id: 'auto-clicker-2',
            name: 'Máy bấm tự động cấp 2',
            cost: 500,
            effect: { type: 'auto-click', value: 5 }
        },
        {
            id: 'click-power-2',
            name: 'Tăng sức mạnh bấm cấp 2',
            cost: 250,
            effect: { type: 'click-power', value: 3 }
        },
    ];

    /**
     * Updates the coin count and UI elements.
     */
    function updateUI() {
        coinDisplay.textContent = coins;
        updateShopItems();
    }

    /**
     * Displays a temporary message box.
     * @param {string} message The message to display.
     */
    function showMessage(message) {
        messageBox.textContent = message;
        messageBox.classList.add('show');
        setTimeout(() => {
            messageBox.classList.remove('show');
        }, 3000); // Hide after 3 seconds
    }

    /**
     * Handles the click event on the main button.
     */
    function handleCoinClick() {
        coins += clickPower;
        updateUI();
    }

    /**
     * Renders all shop items and updates the buy button state.
     */
    function updateShopItems() {
        shopItemsContainer.innerHTML = ''; // Clear previous items

        shopItems.forEach(item => {
            const itemElement = document.createElement('div');
            itemElement.className = 'shop-item';

            itemElement.innerHTML = `
                <div>
                    <h3 class="font-semibold">${item.name}</h3>
                    <p class="text-sm text-gray-400">Giá: <span class="text-yellow-300 font-bold">${item.cost}</span></p>
                </div>
                <button data-item-id="${item.id}" ${coins < item.cost ? 'disabled' : ''}>
                    Mua
                </button>
            `;

            const buyButton = itemElement.querySelector('button');
            buyButton.addEventListener('click', () => buyItem(item.id));

            shopItemsContainer.appendChild(itemElement);
        });
    }

    /**
     * Handles the purchase of an item.
     * @param {string} itemId The ID of the item to buy.
     */
    function buyItem(itemId) {
        const item = shopItems.find(i => i.id === itemId);

        if (!item) {
            console.error('Item not found:', itemId);
            return;
        }

        if (coins >= item.cost) {
            coins -= item.cost;

            // Apply the item's effect
            if (item.effect.type === 'auto-click') {
                autoClickerRate += item.effect.value;
                showMessage(`Bạn đã mua ${item.name}! Tốc độ tự động tăng thêm ${item.effect.value} mỗi giây.`);
            } else if (item.effect.type === 'click-power') {
                clickPower += item.effect.value;
                showMessage(`Bạn đã mua ${item.name}! Sức mạnh bấm tăng thêm ${item.effect.value}.`);
            }

            // A small hack to remove the purchased item from the shop for this example
            const index = shopItems.findIndex(i => i.id === itemId);
            if (index !== -1) {
                shopItems.splice(index, 1);
            }

            // Restart the auto-clicker loop to apply the new rate
            startAutoClicker();
            updateUI();
        } else {
            showMessage('Không đủ tiền!');
        }
    }

    /**
     * Starts the auto-clicker game loop.
     */
    function startAutoClicker() {
        // Clear any existing interval to prevent multiple loops
        if (autoClickerInterval) {
            clearInterval(autoClickerInterval);
        }

        autoClickerInterval = setInterval(() => {
            if (autoClickerRate > 0) {
                coins += autoClickerRate;
                updateUI();
            }
        }, 1000); // Adds coins every 1 second
    }

    // Add event listeners
    window.onload = function() {
        clickButton.addEventListener('click', handleCoinClick);
        
        shopButton.addEventListener('click', () => {
            shopModal.classList.remove('hidden');
            updateShopItems(); // Ensure shop items are up-to-date
        });

        closeShopButton.addEventListener('click', () => {
            shopModal.classList.add('hidden');
        });
        
        // Initial setup
        updateUI();
        startAutoClicker();
    }
</script>
</body>
</html>
