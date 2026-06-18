# novatrust-bank
server.js
package.json
db.json
/public/index.html
/public/dashboard.html
/public/admin.html
{
  "name": "novatrust-bank",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
const express = require("express");
const fs = require("fs");
const app = express();

app.use(express.json());
app.use(express.static("public"));

const DB_FILE = "./db.json";

function readDB() {
  return JSON.parse(fs.readFileSync(DB_FILE));
}

function writeDB(data) {
  fs.writeFileSync(DB_FILE, JSON.stringify(data, null, 2));
}

// LOGIN
app.post("/login", (req, res) => {
  const db = readDB();
  const user = db.users.find(u => u.username === req.body.username);

  if (user && user.password === req.body.password) {
    res.json({ success: true, role: user.role });
  } else {
    res.json({ success: false });
  }
});

// GET BALANCE
app.get("/balance/:username", (req, res) => {
  const db = readDB();
  const user = db.users.find(u => u.username === req.params.username);
  res.json({ balance: user.balance });
});

// ADMIN ADD MONEY
app.post("/admin/add", (req, res) => {
  const db = readDB();
  const user = db.users.find(u => u.username === req.body.username);

  user.balance += Number(req.body.amount);

  writeDB(db);

  res.json({ success: true });
});

app.listen(process.env.PORT || 3000, () => {
  console.log("NovaTrust Bank running...");
});
{
  "users": [
    {
      "username": "user1",
      "password": "1234",
      "role": "user",
      "balance": 1000
    },
    {
      "username": "admin",
      "password": "admin123",
      "role": "admin",
      "balance": 0
    }
  ]
}
<h2>NovaTrust Bank Login</h2>

<input id="u" placeholder="username">
<input id="p" type="password" placeholder="password">

<button onclick="login()">Login</button>

<script>
function login(){
  fetch("/login", {
    method:"POST",
    headers: {"Content-Type":"application/json"},
    body: JSON.stringify({
      username: u.value,
      password: p.value
    })
  })
  .then(r => r.json())
  .then(data => {
    if(data.success){
      if(data.role === "admin"){
        window.location = "admin.html";
      } else {
        window.location = "dashboard.html?user=" + u.value;
      }
    } else {
      alert("Invalid login");
    }
  });
}
</script>
<h2>User Dashboard</h2>

<h3 id="bal">Loading...</h3>

<script>
const user = new URLSearchParams(location.search).get("user");

fetch("/balance/" + user)
.then(r => r.json())
.then(data => {
  document.getElementById("bal").innerText =
  "Balance: $" + data.balance;
});
</script>
<h2>Admin Panel</h2>

<input id="user" placeholder="username">
<input id="amount" placeholder="amount">

<button onclick="addMoney()">Add Money</button>

<script>
function addMoney(){
  fetch("/admin/add", {
    method:"POST",
    headers: {"Content-Type":"application/json"},
    body: JSON.stringify({
      username: user.value,
      amount: amount.value
    })
  })
  .then(() => alert("Updated successfully"));
}
</script>
