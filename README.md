# sip-brew
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sip & Brew - Advanced POS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .menu-item {
            transition: all 0.2s ease-in-out;
        }
        .menu-item:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
        }
        .category-header {
            cursor: pointer;
        }
        .bill-table-row:nth-child(even) {
            background-color: #f9fafb;
        }
        #payment-modal, #whatsapp-modal {
            transition: opacity 0.3s ease;
        }
        .sale-item:nth-child(even){
             background-color: #f9fafb;
        }
        .in-process-item {
            transition: background-color 0.2s ease-in-out;
        }
        /* Hide print area by default on screen */
        .print-area {
            display: none;
        }
        
        @media print {
            /* Hide everything in the body by default when printing */
            body * {
                visibility: hidden;
            }
            /* Make the print-area and its children visible */
            .print-area, .print-area * {
                visibility: visible;
            }
            /* Ensure the print-area is displayed as a block and positioned correctly */
            .print-area {
                display: block;
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
            }
            /* Hide elements that should not be printed */
            .no-print {
                display: none;
            }
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div class="container mx-auto p-4 md:p-6 lg:p-8">
        <header class="text-center mb-8">
            <h1 class="text-4xl font-bold text-amber-800">SIP & BREW</h1>
            <p class="text-lg text-gray-600">Advanced Point of Sale System</p>
        </header>

        <div class="flex flex-col lg:flex-row gap-8">
            <div id="menu-container" class="w-full lg:w-3/5 space-y-4 no-print">
                </div>

            <div class="w-full lg:w-2/5 space-y-8 no-print">
                <div class="bg-white rounded-lg shadow-lg p-6 sticky top-6">
                    <h2 class="text-2xl font-bold border-b pb-3 mb-4 text-amber-900">Current Order</h2>
                    
                    <div id="bill-items" class="max-h-64 overflow-y-auto pr-2 mb-4">
                        <p id="empty-bill-message" class="text-gray-500 text-center py-8">Start a new order.</p>
                    </div>

                    <div class="space-y-3 border-t pt-4">
                        <div class="flex justify-between items-center text-lg"><span class="font-semibold">Subtotal</span><span id="subtotal" class="font-bold">₹0.00</span></div>
                        <div class="flex justify-between items-center"><label for="discount" class="font-semibold">Discount (%)</label><input type="number" id="discount" value="0" min="0" max="100" class="w-20 rounded-md border-gray-300 shadow-sm focus:border-amber-500 focus:ring-amber-500 text-right font-semibold"></div>
                        <div class="flex justify-between items-center text-red-600"><span class="font-semibold">Discount Amount</span><span id="discount-amount" class="font-bold">- ₹0.00</span></div>
                        <div class="flex justify-between items-center text-2xl font-bold text-amber-900 border-t-2 border-dashed pt-3 mt-3"><span>Total</span><span id="total">₹0.00</span></div>
                    </div>
                    
                    <div class="mt-6 space-y-3">
                        <button id="add-to-sale" class="w-full bg-green-600 text-white py-3 rounded-lg font-semibold hover:bg-green-700 transition-colors text-lg">Complete Sale</button>
                        <div class="grid grid-cols-4 gap-3">
                           <button id="hold-order" class="bg-yellow-500 text-white py-2 rounded-lg font-semibold hover:bg-yellow-600 transition-colors">Hold</button>
                           <button id="print-current-bill" class="bg-blue-500 text-white py-2 rounded-lg font-semibold hover:bg-blue-600 transition-colors">Print</button>
                           <button id="whatsapp-bill" class="bg-green-500 text-white py-2 rounded-lg font-semibold hover:bg-green-600 transition-colors flex items-center justify-center gap-1">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 24 24" fill="currentColor"><path d="M.057 24l1.687-6.163c-1.041-1.804-1.588-3.849-1.587-5.946.003-6.556 5.338-11.891 11.893-11.891 3.181.001 6.167 1.24 8.413 3.488 2.245 2.248 3.481 5.236 3.48 8.414-.003 6.557-5.338 11.892-11.894 11.892-1.99-.001-3.951-.5-5.688-1.448l-6.305 1.654zm6.597-3.807c1.676.995 3.276 1.591 5.392 1.592 5.448 0 9.886-4.434 9.889-9.885.002-5.462-4.415-9.89-9.881-9.892-5.452 0-9.887 4.434-9.889 9.886-.001 2.267.651 4.383 1.803 6.14l-1.341 4.894 5.069-1.339z"/></svg>
                                Send
                           </button>
                           <button id="new-order" class="bg-gray-500 text-white py-2 rounded-lg font-semibold hover:bg-gray-600 transition-colors">New</button>
                        </div>
                    </div>
                </div>

                <div class="bg-white rounded-lg shadow-lg p-6">
                    <h2 class="text-2xl font-bold border-b pb-3 mb-4 text-amber-900">In-Process Orders</h2>
                    <div id="in-process-list" class="max-h-96 overflow-y-auto pr-2">
                        <p id="no-in-process-message" class="text-gray-500 text-center py-8">No orders on hold.</p>
                    </div>
                </div>

                <div class="bg-white rounded-lg shadow-lg p-6">
                    <div class="flex justify-between items-center border-b pb-3 mb-4">
                        <h2 class="text-2xl font-bold text-amber-900">Today's Sales</h2>
                        <button id="print-daily-report" class="no-print bg-indigo-600 text-white px-4 py-2 rounded-lg font-semibold hover:bg-indigo-700 transition-colors text-sm">Print Daily Report</button>
                    </div>
                    <div class="flex justify-between items-center bg-amber-100 p-4 rounded-lg mb-4">
                        <span class="text-xl font-bold text-amber-800">Total Sales</span>
                        <span id="total-sales" class="text-2xl font-extrabold text-amber-900">₹0.00</span>
                    </div>
                    <div id="sales-list" class="max-h-96 overflow-y-auto pr-2">
                        <p id="no-sales-message" class="text-gray-500 text-center py-8">No sales recorded yet.</p>
                    </div>
                </div>

                <div class="bg-white rounded-lg shadow-lg p-6">
                    <h2 class="text-2xl font-bold border-b pb-3 mb-4 text-amber-900">Items Sold Today</h2>
                    <div id="item-summary-list" class="max-h-96 overflow-y-auto pr-2">
                         <p id="no-items-sold-message" class="text-gray-500 text-center py-8">No items have been sold yet.</p>
                    </div>
                </div>

            </div>
        </div>
    </div>
    
    <div id="payment-modal" class="no-print fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 hidden z-50">
        <div class="bg-white rounded-lg shadow-xl p-8 max-w-sm w-full">
            <h3 class="text-2xl font-bold text-center mb-6">Select Payment Method</h3>
            <div class="space-y-4">
                <button onclick="processSale('Cash')" class="w-full text-lg bg-emerald-500 text-white py-4 rounded-lg font-semibold hover:bg-emerald-600 transition-colors">Cash</button>
                <button onclick="processSale('UPI')" class="w-full text-lg bg-blue-500 text-white py-4 rounded-lg font-semibold hover:bg-blue-600 transition-colors">UPI</button>
                <button onclick="processSale('Online Order')" class="w-full text-lg bg-orange-500 text-white py-4 rounded-lg font-semibold hover:bg-orange-600 transition-colors">Online Order</button>
            </div>
            <button onclick="hidePaymentModal()" class="mt-6 w-full text-gray-600 hover:text-gray-800 font-semibold">Cancel</button>
        </div>
    </div>

    <div id="whatsapp-modal" class="no-print fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 hidden z-50">
        <div class="bg-white rounded-lg shadow-xl p-8 max-w-sm w-full">
            <h3 class="text-2xl font-bold text-center mb-4">Send Bill via WhatsApp</h3>
            <div class="space-y-4">
                <div>
                    <label for="whatsapp-phone-number" class="block text-sm font-medium text-gray-700 mb-1">Customer's Phone Number</label>
                    <div class="flex">
                        <input type="text" id="whatsapp-country-code" value="+91" class="w-16 text-center px-3 py-2 border border-gray-300 rounded-l-md shadow-sm focus:outline-none focus:ring-amber-500 focus:border-amber-500">
                        <input type="tel" id="whatsapp-phone-number" placeholder="9876543210" class="w-full px-3 py-2 border border-l-0 border-gray-300 rounded-r-md shadow-sm focus:outline-none focus:ring-amber-500 focus:border-amber-500">
                    </div>
                </div>
                <button id="send-whatsapp-btn" class="w-full text-lg bg-green-500 text-white py-3 rounded-lg font-semibold hover:bg-green-600 transition-colors">Send</button>
            </div>
            <button id="cancel-whatsapp-btn" class="mt-6 w-full text-gray-600 hover:text-gray-800 font-semibold">Cancel</button>
        </div>
    </div>

    <div class="print-area"></div>


    <script>
    // --- DATA --- (Updated from your CSV file)
    const menuData = {"Coffee":[{"name":"Iced Latte","prices":{"Reg":99,"Lrg":119}},{"name":"Iced Caramel Coffee","prices":{"Reg":139,"Lrg":169}},{"name":"Cappuccino","prices":{"Reg":79,"Lrg":99}},{"name":"Iced Coffee","prices":{"Reg":99,"Lrg":119}},{"name":"Iced Chocolate Mocha","prices":{"Reg":139,"Lrg":169}},{"name":"Vanilla Mist Latte","prices":{"Reg":149,"Lrg":179}},{"name":"Espresso Coffee","prices":{"Hot":79,"Cold":99}},{"name":"Spanish Iced Latte","prices":{"Reg":169,"Lrg":189}},{"name":"Cold Frappe","prices":{"Reg":169,"Lrg":189}}],"Shakes":[{"name":"Oreo Shake","prices":{"Regular":149}},{"name":"Strawberry Shake","prices":{"Regular":129}},{"name":"Blueberry Shake","prices":{"Regular":119}},{"name":"Black Currant Shake","prices":{"Regular":119}}],"Hot Beverages":[{"name":"Honey Ginger Tea","prices":{"Regular":79}},{"name":"Milk Tea (Masala, Ginger, Cardamom)","prices":{"Regular":49}}],"Cold Beverages":[{"name":"Virgin Mojito","prices":{"Regular":99}},{"name":"Peach Ice Tea","prices":{"Regular":119}},{"name":"Blue Curacao","prices":{"Regular":119}},{"name":"Fresh Lime Soda","prices":{"Regular":69}}],"Sides":[{"name":"French Fries","prices":{"Regular":69}},{"name":"Peri Peri Fries","prices":{"Regular":79}},{"name":"Loaded Puff","prices":{"Regular":69}},{"name":"Cheesy Loaded Puff","prices":{"Regular":99}},{"name":"Cheesy Garlic Bread","prices":{"Regular":99}},{"name":"Cheese Sticks","prices":{"Regular":149}}],"Sandwiches":[{"name":"Indian Veggie Sandwich","prices":{"Regular":89}},{"name":"American Veggie Sandwich","prices":{"Regular":99}},{"name":"Paneer Tikka Sandwich","prices":{"Regular":119}},{"name":"Cheese Corn Veg Sandwich","prices":{"Regular":129}},{"name":"Cheesy Grill Sandwich","prices":{"Regular":149}}],"Burgers":[{"name":"Aloo Tikki Burger","prices":{"Regular":69}},{"name":"Paneer Tikki Burger","prices":{"Regular":99}},{"name":"Cheesy Fusion Burger","prices":{"Regular":119}}],"Wraps":[{"name":"Aloo Tikki Wrap","prices":{"Regular":79}},{"name":"Tandoori Paneer Wrap","prices":{"Regular":99}},{"name":"Crispy Paneer Wrap","prices":{"Regular":119}}],"Pasta":[{"name":"White Sauce Pasta","prices":{"Regular":129}},{"name":"Red Sauce Pasta","prices":{"Regular":139}},{"name":"Mix Sauce Pasta","prices":{"Regular":149}}],"Pizza":[{"name":"Margherita Pizza","prices":{"Regular":99}},{"name":"Sweet Corn Pizza","prices":{"Regular":119}},{"name":"Farm House Pizza","prices":{"Regular":149}},{"name":"Paneer Tikka Pizza","prices":{"Regular":179}}],"Add-ons":[{"name":"Flavor (Vanilla, Choc, Hazel, Caramel)","prices":{"Add-on":30}},{"name":"Cheese Burst","prices":{"Add-on":50}},{"name":"Cheese Slice","prices":{"Add-on":29}},{"name":"Extra Tikki","prices":{"Add-on":29}},{"name":"Dip (Any)","prices":{"Add-on":20}}]};
    
    // --- IMPORTANT: Add your QR Code Image URL here ---
    // Upload your QR code image to a hosting service (like imgur.com) and paste the direct link below.
    const qrCodeImageUrl = "https://i.imgur.com/g8D22sL.png"; // Example QR code, replace with your own URL

    let currentOrder = { items: [], discount: 0 };
    let inProcessOrders = [];
    let salesData = [];
// ... existing code ... -->
            let message = `*Your Bill from SIP & BREW*\n\n`;
            message += `------------------------------------\n`;
            order.items.forEach(item => {
                const itemTotal = (item.price * item.quantity).toFixed(2);
                message += `${item.name} (${item.size}) x ${item.quantity} - ₹${itemTotal}\n`;
            });
            message += `------------------------------------\n`;
            message += `*Subtotal:* ₹${subtotal.toFixed(2)}\n`;
            if (order.discount > 0) {
                message += `*Discount (${order.discount}%):* -₹${discountAmount.toFixed(2)}\n`;
            }
            message += `*TOTAL: ₹${total.toFixed(2)}*\n\n`;
            message += `Thank you for your visit!`;
            
            // Add QR code link to the message if it exists
            if (qrCodeImageUrl && qrCodeImageUrl !== "https://i.imgur.com/g8D22sL.png") {
                message += `\n\nScan QR to Pay: ${qrCodeImageUrl}`;
            }

            return message;
        }

        function sendBillToWhatsapp() {
// ... existing code ... -->
        function getReceiptHTML(billNo, timestamp, items, discountPercent, paymentMethod) {
            const subtotal = items.reduce((acc, item) => acc + (item.price * item.quantity), 0);
            const discountAmount = subtotal * (discountPercent / 100);
            const total = subtotal - discountAmount;
            
            let qrCodeHtml = '';
            // Add QR code image to the receipt if the URL is provided
            if (qrCodeImageUrl && qrCodeImageUrl !== "https://i.imgur.com/g8D22sL.png") {
                qrCodeHtml = `<div style="text-align: center; margin-top: 15px;">
                                <p style="font-size: 12px; margin: 0;">Scan to Pay</p>
                                <img src="${qrCodeImageUrl}" alt="QR Code for Payment" style="width: 120px; height: 120px; margin: 5px auto 0;">
                              </div>`;
            }

            return ` <div style="font-family: 'Courier New', monospace; width: 300px; margin: 0 auto; padding: 20px; border: 1px solid #ccc;"> 
                        <h2 style="text-align: center; margin:0;">SIP & BREW</h2> 
                        <p style="text-align: center; font-size: 12px; border-bottom: 1px dashed #000; padding-bottom: 10px; margin-bottom:10px;"> Bill #: ${billNo} | ${timestamp.toLocaleDateString()} ${timestamp.toLocaleTimeString()} </p> 
                        <table style="width: 100%; font-size: 12px; border-collapse: collapse;"> 
                            <thead><tr><th style="text-align: left;">Item</th><th style="text-align: center;">Qty</th><th style="text-align: right;">Total</th></tr></thead> 
                            <tbody>${items.map(item => `<tr><td>${item.name} (${item.size})</td><td style="text-align: center;">${item.quantity}</td><td style="text-align: right;">${(item.quantity * item.price).toFixed(2)}</td></tr>`).join('')}</tbody> 
                        </table> 
                        <hr style="border-top: 1px dashed #000; margin: 10px 0;"> 
                        <div style="font-size: 14px;"> 
                            <p style="display: flex; justify-content: space-between;"><strong>Subtotal:</strong> <span>₹${subtotal.toFixed(2)}</span></p> 
                            <p style="display: flex; justify-content: space-between;"><strong>Discount (${discountPercent}%):</strong> <span>- ₹${discountAmount.toFixed(2)}</span></p> 
                            <hr style="border-top: 1px dashed #000; margin: 10px 0;"> 
                            <p style="display: flex; justify-content: space-between; font-size: 18px;"><strong>TOTAL:</strong> <strong>₹${total.toFixed(2)}</strong></p> 
                            <p style="display: flex; justify-content: space-between;"><strong>Payment:</strong> <span>${paymentMethod}</span></p> 
                        </div> 
                        <p style="text-align: center; font-size: 12px; margin-top: 20px;">Thank You!</p> 
                        ${qrCodeHtml}
                     </div>`; 
        }

        discountEl.addEventListener('input', calculateTotals);
        newOrderBtn.addEventListener('click', () => { if(currentOrder.items.length > 0) holdCurrentOrder(); resetCurrentOrder(); });
// ... existing code ... -->

