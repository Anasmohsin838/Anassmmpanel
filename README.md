<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Anas SMM Panel</title>
  <style>
    body {
      font-family: sans-serif;
      background: #f0f0f0;
      padding: 20px;
      text-align: center;
    }
    .box {
      background: white;
      border-radius: 12px;
      padding: 20px;
      margin: auto;
      max-width: 500px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    input, select {
      width: 90%;
      padding: 10px;
      margin: 8px 0;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
    button {
      background: #25D366;
      color: white;
      border: none;
      padding: 12px 25px;
      font-size: 16px;
      border-radius: 8px;
      cursor: pointer;
      margin: 10px 0;
    }
    .hidden { display: none; }
    .order-list {
      text-align: left;
      margin-top: 20px;
    }
    .order-list li {
      margin: 5px 0;
    }
    #liveChat {
      position: fixed;
      bottom: 20px;
      right: 20px;
      background: #25D366;
      color: white;
      border: none;
      border-radius: 50%;
      width: 60px;
      height: 60px;
      font-size: 30px;
      cursor: pointer;
      box-shadow: 0 0 10px rgba(0,0,0,0.3);
    }
  </style>
</head>
<body>

<div class="box" id="authBox">
  <h2>🔐 Anas SMM Panel</h2>
  <input type="email" id="email" placeholder="Email" required />
  <input type="password" id="password" placeholder="Password" required />
  <button onclick="register()">Sign Up</button>
  <button onclick="login()">Login</button>
</div>

<div class="box hidden" id="mainPanel">
  <h3>🎉 Welcome to Anas SMM Panel</h3>
  <p><strong>Current Balance:</strong> Rs. <span id="balance">0</span></p>
  <p><strong>Total Spent:</strong> Rs. <span id="spent">0</span></p>
  <p><strong>Total Orders:</strong> <span id="orders">0</span></p>

  <button onclick="addFunds()">💸 Add Funds</button>

  <form onsubmit="placeOrder(event)">
    <h4>🛒 Place New Order</h4>
    <select id="service" required>
      <option value="">Select Service</option>
      <option value="TikTok Likes">TikTok Likes - Rs 0.1 per unit</option>
      <option value="TikTok Views">TikTok Views - Rs 0.02 per unit</option>
      <option value="TikTok Followers">TikTok Followers - Rs 0.55 per unit</option>
      <option value="TikTok Shares">TikTok Shares - Rs 0.03 per unit</option>
      <option value="Instagram Followers">Instagram Followers - Rs 0.4 per unit</option>
      <option value="Instagram Likes">Instagram Likes - Rs 0.12 per unit</option>
      <option value="Instagram Views">Instagram Views - Rs 0.22 per unit</option>
      <option value="Facebook Followers">Facebook Followers - Rs 0.4 per unit</option>
      <option value="Facebook Likes">Facebook Likes - Rs 0.3 per unit</option>
      <option value="Facebook Views">Facebook Views - Rs 0.12 per unit</option>
      <option value="YouTube Subscribers">YouTube Subscribers - Rs 0.8 per unit</option>
      <option value="YouTube Likes">YouTube Likes - Rs 0.3 per unit</option>
      <option value="YouTube Views">YouTube Views - Rs 0.45 per unit</option>
      <option value="YouTube Watch Time">YouTube Watch Time - Rs 9 per unit</option>
    </select>
    <input type="number" id="qty" placeholder="Quantity" required min="1" max="20000000" />
    <input type="text" id="link" placeholder="Video/Profile Link" required />
    <p>Total Price: Rs. <span id="price">0</span></p>
    <button type="submit">Place Order</button>
  </form>

  <div class="order-list">
    <h4>📋 My Orders</h4>
    <ul id="orderList"></ul>
  </div>

  <button onclick="logout()">🚪 Logout</button>
</div>

<button id="liveChat" title="Chat with us" onclick="liveChat()">💬</button>

<script>
  const prices = {
    "TikTok Likes": 0.1,
    "TikTok Views": 0.02,
    "TikTok Followers": 0.55,
    "TikTok Shares": 0.03,
    "Instagram Followers": 0.4,
    "Instagram Likes": 0.12,
    "Instagram Views": 0.22,
    "Facebook Followers": 0.4,
    "Facebook Likes": 0.3,
    "Facebook Views": 0.12,
    "YouTube Subscribers": 0.8,
    "YouTube Likes": 0.3,
    "YouTube Views": 0.45,
    "YouTube Watch Time": 9
  };

  let user = null;

  function register() {
    const email = document.getElementById("email").value.trim();
    const pass = document.getElementById("password").value.trim();
    if (!email || !pass) {
      alert("Please enter email and password");
      return;
    }
    if (localStorage.getItem(email)) {
      alert("Account already exists. Please login.");
      return;
    }
    const data = {
      password: pass,
      balance: 0,
      spent: 0,
      orders: 0,
      orderList: []
    };
    localStorage.setItem(email, JSON.stringify(data));
    alert("Account created! Please login.");
    // Auto login after register
    user = email;
    showPanel(data);
  }

  function login() {
    const email = document.getElementById("email").value.trim();
    const pass = document.getElementById("password").value.trim();
    if (!email || !pass) {
      alert("Please enter email and password");
      return;
    }
    const data = localStorage.getItem(email);
    if (!data) {
      alert("Account not found! Please register.");
      return;
    }
    const userData = JSON.parse(data);
    if (userData.password !== pass) {
      alert("Wrong password!");
      return;
    }
    user = email;
    showPanel(userData);
  }

  function showPanel(data) {
    document.getElementById("authBox").classList.add("hidden");
    document.getElementById("mainPanel").classList.remove("hidden");
    document.getElementById("balance").innerText = data.balance.toFixed(2);
    document.getElementById("spent").innerText = data.spent.toFixed(2);
    document.getElementById("orders").innerText = data.orders;
    renderOrders(data.orderList || []);
  }

  function logout() {
    user = null;
    document.getElementById("mainPanel").classList.add("hidden");
    document.getElementById("authBox").classList.remove("hidden");
    document.getElementById("email").value = "";
    document.getElementById("password").value = "";
  }

  function calcPrice() {
    const service = document.getElementById("service").value;
    const qty = parseFloat(document.getElementById("qty").value || 0);
    const rate = prices[service] || 0;
    document.getElementById("price").innerText = (qty * rate).toFixed(2);
  }

  document.getElementById("qty").addEventListener("input", calcPrice);
  document.getElementById("service").addEventListener("change", calcPrice);

  function addFunds() {
    alert(
      "JazzCash Payment Details:\n" +
      "Account Number: 03270823585\n" +
      "Account Holder: Fazila Mazhar\n\n" +
      "Please send the amount via JazzCash and then message on WhatsApp to update your balance."
    );
  }

  function placeOrder(e) {
    e.preventDefault();
    if (!user) {
      alert("Please login first.");
      return;
    }
    const service = document.getElementById("service").value;
    const qty = parseFloat(document.getElementById("qty").value);
    const link = document.getElementById("link").value.trim();
    if (!service || !qty || qty <= 0 || !link) {
      alert("Please fill all order details correctly.");
      return;
    }
    const rate = prices[service];
    const total = qty * rate;

    let data = JSON.parse(localStorage.getItem(user));
    if (data.balance < total) {
      alert("Not enough balance.");
      return;
    }

    const order = {
      id: Date.now(),
      service,
      qty,
      link,
      price: total.toFixed(2)
    };

    data.balance -= total;
    data.spent += total;
    data.orders += 1;
    data.orderList.push(order);

    localStorage.setItem(user, JSON.stringify(data));
    showPanel(data);

    // Send order details to your WhatsApp
    const msg = `📦 New Order:%0AUser: ${user}%0AService: ${service}%0AQty: ${qty}%0ALink: ${encodeURIComponent(link)}%0ATotal: Rs. ${total.toFixed(2)}`;
    window.open(`https://wa.me/923270823585?text=${msg}`, "_blank");

    // Reset form
    document.getElementById("service").value = "";
    document.getElementById("qty").value = "";
    document.getElementById("link").value = "";
    document.getElementById("price").innerText = "0";
  }

  function renderOrders(orders) {
    const list = document.getElementById("orderList");
    list.innerHTML = "";
    orders.forEach(order => {
      const li = document.createElement("li");
      li.innerHTML = `${order.service} (${order.qty}) - Rs. ${order.price} 
      <button onclick="cancelOrder(${order.id})" style="background:red; margin-left:10px; cursor:pointer;">Cancel</button>`;
      list.appendChild(li);
    });
  }

  function cancelOrder(orderId) {
    if (!user) return;
    let data = JSON.parse(localStorage.getItem(user));
    const index = data.orderList.findIndex(o => o.id === orderId);
    if (index !== -1) {
      const refund = parseFloat(data.orderList[index].price);
      data.balance += refund;
      data.spent -= refund;
      data.orders -= 1;
      data.orderList.splice(index, 1);
      localStorage.setItem(user, JSON.stringify(data));
      showPanel(data);
      alert("Order cancelled & amount refunded.");
    }
  }

  function liveChat() {
    // Open WhatsApp chat for live support
    window.open("https://wa.me/923270823585", "_blank");
  }
</script>

</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Anas SMM Panel</title>
  <style>
    body {
      font-family: sans-serif;
      background: #f0f0f0;
      padding: 20px;
      text-align: center;
    }
    .box {
      background: white;
      border-radius: 12px;
      padding: 20px;
      margin: auto;
      max-width: 500px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    input, select {
      width: 90%;
      padding: 10px;
      margin: 8px 0;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
    button {
      background: #25D366;
      color: white;
      border: none;
      padding: 12px 25px;
      font-size: 16px;
      border-radius: 8px;
      cursor: pointer;
      margin: 10px 0;
    }
    .hidden { display: none; }
    .order-list {
      text-align: left;
      margin-top: 20px;
    }
    .order-list li {
      margin: 5px 0;
    }
    #liveChat {
      position: fixed;
      bottom: 20px;
      right: 20px;
      background: #25D366;
      color: white;
      border: none;
      border-radius: 50%;
      width: 60px;
      height: 60px;
      font-size: 30px;
      cursor: pointer;
      box-shadow: 0 0 10px rgba(0,0,0,0.3);
    }
  </style>
</head>
<body>

<div class="box" id="authBox">
  <h2>🔐 Anas SMM Panel</h2>
  <input type="email" id="email" placeholder="Email" required />
  <input type="password" id="password" placeholder="Password" required />
  <button onclick="register()">Sign Up</button>
  <button onclick="login()">Login</button>
</div>

<div class="box hidden" id="mainPanel">
  <h3>🎉 Welcome to Anas SMM Panel</h3>
  <p><strong>Current Balance:</strong> Rs. <span id="balance">0</span></p>
  <p><strong>Total Spent:</strong> Rs. <span id="spent">0</span></p>
  <p><strong>Total Orders:</strong> <span id="orders">0</span></p>

  <button onclick="addFunds()">💸 Add Funds</button>

  <form onsubmit="placeOrder(event)">
    <h4>🛒 Place New Order</h4>
    <select id="service" required>
      <option value="">Select Service</option>
      <option value="TikTok Likes">TikTok Likes - Rs 0.1 per unit</option>
      <option value="TikTok Views">TikTok Views - Rs 0.02 per unit</option>
      <option value="TikTok Followers">TikTok Followers - Rs 0.55 per unit</option>
      <option value="TikTok Shares">TikTok Shares - Rs 0.03 per unit</option>
      <option value="Instagram Followers">Instagram Followers - Rs 0.4 per unit</option>
      <option value="Instagram Likes">Instagram Likes - Rs 0.12 per unit</option>
      <option value="Instagram Views">Instagram Views - Rs 0.22 per unit</option>
      <option value="Facebook Followers">Facebook Followers - Rs 0.4 per unit</option>
      <option value="Facebook Likes">Facebook Likes - Rs 0.3 per unit</option>
      <option value="Facebook Views">Facebook Views - Rs 0.12 per unit</option>
      <option value="YouTube Subscribers">YouTube Subscribers - Rs 0.8 per unit</option>
      <option value="YouTube Likes">YouTube Likes - Rs 0.3 per unit</option>
      <option value="YouTube Views">YouTube Views - Rs 0.45 per unit</option>
      <option value="YouTube Watch Time">YouTube Watch Time - Rs 9 per unit</option>
    </select>
    <input type="number" id="qty" placeholder="Quantity" required min="1" max="20000000" />
    <input type="text" id="link" placeholder="Video/Profile Link" required />
    <p>Total Price: Rs. <span id="price">0</span></p>
    <button type="submit">Place Order</button>
  </form>

  <div class="order-list">
    <h4>📋 My Orders</h4>
    <ul id="orderList"></ul>
  </div>

  <button onclick="logout()">🚪 Logout</button>
</div>

<button id="liveChat" title="Chat with us" onclick="liveChat()">💬</button>

<script>
  const prices = {
    "TikTok Likes": 0.1,
    "TikTok Views": 0.02,
    "TikTok Followers": 0.55,
    "TikTok Shares": 0.03,
    "Instagram Followers": 0.4,
    "Instagram Likes": 0.12,
    "Instagram Views": 0.22,
    "Facebook Followers": 0.4,
    "Facebook Likes": 0.3,
    "Facebook Views": 0.12,
    "YouTube Subscribers": 0.8,
    "YouTube Likes": 0.3,
    "YouTube Views": 0.45,
    "YouTube Watch Time": 9
  };

  let user = null;

  function register() {
    const email = document.getElementById("email").value.trim();
    const pass = document.getElementById("password").value.trim();
    if (!email || !pass) {
      alert("Please enter email and password");
      return;
    }
    if (localStorage.getItem(email)) {
      alert("Account already exists. Please login.");
      return;
    }
    const data = {
      password: pass,
      balance: 0,
      spent: 0,
      orders: 0,
      orderList: []
    };
    localStorage.setItem(email, JSON.stringify(data));
    alert("Account created! Please login.");
    // Auto login after register
    user = email;
    showPanel(data);
  }

  function login() {
    const email = document.getElementById("email").value.trim();
    const pass = document.getElementById("password").value.trim();
    if (!email || !pass) {
      alert("Please enter email and password");
      return;
    }
    const data = localStorage.getItem(email);
    if (!data) {
      alert("Account not found! Please register.");
      return;
    }
    const userData = JSON.parse(data);
    if (userData.password !== pass) {
      alert("Wrong password!");
      return;
    }
    user = email;
    showPanel(userData);
  }

  function showPanel(data) {
    document.getElementById("authBox").classList.add("hidden");
    document.getElementById("mainPanel").classList.remove("hidden");
    document.getElementById("balance").innerText = data.balance.toFixed(2);
    document.getElementById("spent").innerText = data.spent.toFixed(2);
    document.getElementById("orders").innerText = data.orders;
    renderOrders(data.orderList || []);
  }

  function logout() {
    user = null;
    document.getElementById("mainPanel").classList.add("hidden");
    document.getElementById("authBox").classList.remove("hidden");
    document.getElementById("email").value = "";
    document.getElementById("password").value = "";
  }

  function calcPrice() {
    const service = document.getElementById("service").value;
    const qty = parseFloat(document.getElementById("qty").value || 0);
    const rate = prices[service] || 0;
    document.getElementById("price").innerText = (qty * rate).toFixed(2);
  }

  document.getElementById("qty").addEventListener("input", calcPrice);
  document.getElementById("service").addEventListener("change", calcPrice);

  function addFunds() {
    alert(
      "JazzCash Payment Details:\n" +
      "Account Number: 03270823585\n" +
      "Account Holder: Fazila Mazhar\n\n" +
      "Please send the amount via JazzCash and then message on WhatsApp to update your balance."
    );
  }

  function placeOrder(e) {
    e.preventDefault();
    if (!user) {
      alert("Please login first.");
      return;
    }
    const service = document.getElementById("service").value;
    const qty = parseFloat(document.getElementById("qty").value);
    const link = document.getElementById("link").value.trim();
    if (!service || !qty || qty <= 0 || !link) {
      alert("Please fill all order details correctly.");
      return;
    }
    const rate = prices[service];
    const total = qty * rate;

    let data = JSON.parse(localStorage.getItem(user));
    if (data.balance < total) {
      alert("Not enough balance.");
      return;
    }

    const order = {
      id: Date.now(),
      service,
      qty,
      link,
      price: total.toFixed(2)
    };

    data.balance -= total;
    data.spent += total;
    data.orders += 1;
    data.orderList.push(order);

    localStorage.setItem(user, JSON.stringify(data));
    showPanel(data);

    // Send order details to your WhatsApp
    const msg = `📦 New Order:%0AUser: ${user}%0AService: ${service}%0AQty: ${qty}%0ALink: ${encodeURIComponent(link)}%0ATotal: Rs. ${total.toFixed(2)}`;
    window.open(`https://wa.me/923270823585?text=${msg}`, "_blank");

    // Reset form
    document.getElementById("service").value = "";
    document.getElementById("qty").value = "";
    document.getElementById("link").value = "";
    document.getElementById("price").innerText = "0";
  }

  function renderOrders(orders) {
    const list = document.getElementById("orderList");
    list.innerHTML = "";
    orders.forEach(order => {
      const li = document.createElement("li");
      li.innerHTML = `${order.service} (${order.qty}) - Rs. ${order.price} 
      <button onclick="cancelOrder(${order.id})" style="background:red; margin-left:10px; cursor:pointer;">Cancel</button>`;
      list.appendChild(li);
    });
  }

  function cancelOrder(orderId) {
    if (!user) return;
    let data = JSON.parse(localStorage.getItem(user));
    const index = data.orderList.findIndex(o => o.id === orderId);
    if (index !== -1) {
      const refund = parseFloat(data.orderList[index].price);
      data.balance += refund;
      data.spent -= refund;
      data.orders -= 1;
      data.orderList.splice(index, 1);
      localStorage.setItem(user, JSON.stringify(data));
      showPanel(data);
      alert("Order cancelled & amount refunded.");
    }
  }

  function liveChat() {
    // Open WhatsApp chat for live support
    window.open("https://wa.me/923270823585", "_blank");
  }
</script>

</body>
</html>
