<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Paphiri Marketplace</title>

<script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>

<style>
body { font-family: Arial, sans-serif; margin: 0; background: #0b1f3a; color: white; }
header { background: #081a33; padding: 12px 20px; display: flex; justify-content: space-between; align-items: center; }
.logo { display: flex; align-items: center; gap: 10px; }
.logo img { height: 40px; }
.search-bar { padding: 10px; width: 40%; border-radius: 5px; border: none; }
.container { display: flex; }
.sidebar { width: 200px; background: #102a54; padding: 15px; }
.sidebar button { width: 100%; margin-bottom: 10px; padding: 10px; background:#1e3a8a; color:white; border:none; }
.main { flex: 1; padding: 15px; }
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 15px; }
.card { background: #142f5c; padding: 10px; border-radius: 8px; }
.card img { width: 100%; height: 120px; object-fit: cover; }
.card button { width: 100%; margin-top: 5px; background:#2563eb; color:white; border:none; padding:8px; }

.modal, .chat-box, .auth-box { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.7); justify-content:center; align-items:center; }

.modal-content { background:white; color:black; padding:20px; border-radius:10px; width:300px; }

/* LOGIN PANEL DESIGN */
.auth-content {
  background: linear-gradient(135deg, #1e3a8a, #facc15, #ef4444);
  padding: 25px;
  border-radius: 12px;
  width: 300px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.auth-content h3 { text-align:center; color:white; }
.auth-content input {
  padding:10px;
  border:none;
  border-radius:6px;
}
.auth-content button {
  padding:10px;
  border:none;
  border-radius:6px;
  background:#0b1f3a;
  color:white;
  font-weight:bold;
}

.chat-content { background:white; color:black; padding:20px; border-radius:10px; width:300px; }
.chat-messages { height:200px; overflow:auto; border:1px solid #ddd; margin-bottom:10px; padding:5px; }

@media(max-width:768px){
 .container{flex-direction:column;}
 .sidebar{width:100%; display:flex; overflow:auto;}
}
.welcome {
  text-align: center;
  padding: 60px 20px;
  margin-bottom: 30px;
  background: linear-gradient(135deg, #0b1f3a, #142f5c);
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.4);
}

.welcome h1 {
  font-size: 48px;
  line-height: 1.2;
  margin-bottom: 15px;
  color: #facc15;
  font-weight: bold;
}

.welcome h1 span {
  color: #fde047;
  text-shadow: 0 0 10px rgba(250,204,21,0.7);
}

.welcome p {
  font-size: 20px;
  color: #e5e7eb;
  margin-bottom: 20px;
}

.welcome button {
  padding: 12px 25px;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  background: #facc15;
  color: #0b1f3a;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}

.welcome button:hover {
  background: #fde047;
  transform: scale(1.05);
}
.add-btn {
  background: linear-gradient(135deg, #1e3a8a, #facc15);
  color: white;
  font-weight: bold;
  border: none;
  padding: 12px;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0,0,0,0.3);
}

.add-btn:hover {
  transform: translateY(-3px) scale(1.03);
  box-shadow: 0 8px 25px rgba(0,0,0,0.5);
  background: linear-gradient(135deg, #2563eb, #fde047);
}

.add-btn:active {
  transform: scale(0.97);
}
.nav-btn {
  background: linear-gradient(135deg, #1e3a8a, #facc15);
  color: white;
  font-weight: bold;
  border: none;
  padding: 10px;
  border-radius: 10px;
  cursor: pointer;
  margin-bottom: 10px;
  transition: all 0.3s ease;
}

.nav-btn:hover {
  transform: translateY(-2px);
  background: linear-gradient(135deg, #2563eb, #fde047);
}
</style>
</head>
<body>

<header>
  <div class="logo">
    <img src="Img/logo.jpeg">
    <h3>Paphiri Market Place</h3>
  </div>
  <input class="search-bar" placeholder="Search..." onkeyup="searchItems(this.value)">
  <button onclick="openAuth()">Login</button>
</header>

<div class="sidebar">

    <button class="add-btn" onclick="openModal()">
        ➕ Add Listing
    </button>

    <button class="nav-btn" onclick="filterCategory('near')">
        📍 Listings Near Me
    </button>

    <button class="nav-btn" onclick="filterCategory('all')">
        📦 All Listings
    </button>

</div>

  <div class="main">

  
<!-- WELCOME TEXT -->
  
<div class="welcome">
    
<h1>Welcome to Paphiri Marketplace</h1>
    
<p>Buy and Sell Platform around Livingstonia</p>
<p>Fast and Secure.</p>
 
 </div>

  <div class="grid" id="listings">
</div>




<!-- ADD LISTING -->
<div class="modal" id="modal">
  <div class="modal-content">
    <h3>New Listing</h3>
    <input id="title" placeholder="Title">
    <textarea id="description" placeholder="Description"></textarea>
    <input id="price" type="number" placeholder="Price">
    <input id="image" placeholder="Image URL">
    <button onclick="addListing()">Post</button>
  </div>
</div>

<!-- AUTH -->
<div class="auth-box" id="auth">
  <div class="auth-content">
    <h3>Welcome to Paphiri</h3>
    <input id="email" placeholder="Email">
    <input id="password" type="password" placeholder="Password">
    <button onclick="login()">Login</button>
    <button onclick="register()">Register</button>
  </div>
</div>

<!-- CHAT -->
<div class="chat-box" id="chat">
  <div class="chat-content">
    <h3>Chat</h3>
    <div class="chat-messages" id="messages"></div>
    <input id="msgInput" placeholder="Type message">
    <button onclick="sendMessage()">Send</button>
  </div>
</div>

<script>
let listings = [];
let socket = io("http://localhost:5000");
let currentRoom = "";

function fetchListings(){
 fetch("http://localhost:5000/api/listings")
 .then(r=>r.json())
 .then(d=>{ listings=d; displayListings(d); });
}

function displayListings(data){
 let el=document.getElementById("listings"); el.innerHTML="";
 data.forEach(i=>{
  let div=document.createElement("div");
  div.className="card";
  div.innerHTML=`
   <img src="${i.image||'https://via.placeholder.com/200'}">
   <h4>${i.title}</h4>
   <p>${i.description}</p>
   <b>K${i.price}</b>
   <button onclick="openChat('${i._id}')">Chat</button>
  `;
  el.appendChild(div);
 });
}

function searchItems(q){
 displayListings(listings.filter(i=>i.title.toLowerCase().includes(q.toLowerCase())));
}

function filterCategory(c){
 if(c==='all') return displayListings(listings);
 displayListings(listings.filter(i=>i.category===c));
}

function openModal(){ modal.style.display="flex"; }

function addListing(){
 fetch("http://localhost:5000/api/listings",{
  method:"POST",
  headers:{"Content-Type":"application/json"},
  body:JSON.stringify({
   title:title.value,
   description:description.value,
   price:price.value,
   image:image.value
  })
 }).then(()=>{ modal.style.display="none"; fetchListings(); });
}

function openAuth(){ auth.style.display="flex"; }

function login(){
 fetch("http://localhost:5000/api/auth/login",{
  method:"POST",
  headers:{"Content-Type":"application/json"},
  body:JSON.stringify({email:email.value,password:password.value})
 }).then(r=>r.json()).then(d=>{
  localStorage.setItem("token",d.token);
  auth.style.display="none";
 });
}

function register(){
 fetch("http://localhost:5000/api/auth/register",{
  method:"POST",
  headers:{"Content-Type":"application/json"},
  body:JSON.stringify({email:email.value,password:password.value})
 });
}

function openChat(id){
 currentRoom=id;
 socket.emit("join",id);
 chat.style.display="flex";
}

socket.on("message",msg=>{
 let div=document.createElement("div");
 div.textContent=msg;
 messages.appendChild(div);
});

function sendMessage(){
 let msg=msgInput.value;
 socket.emit("message",{room:currentRoom,msg});
 msgInput.value="";
}

fetchListings();
</script>

</body>
</html>
