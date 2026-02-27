<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>C Mart - Grocery Store</title>

<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
<script src="https://cdn.emailjs.com/dist/email.min.js"></script>

<style>
body{
    margin:0;
    font-family:Arial, sans-serif;
    background:#f4f6f9;
}

header{
    background:#0a7c2f;
    color:white;
    padding:15px;
    display:flex;
    justify-content:space-between;
    align-items:center;
}

header h1{margin:0;}

header button{
    background:white;
    color:#0a7c2f;
    border:none;
    padding:8px 12px;
    border-radius:5px;
    cursor:pointer;
    font-weight:bold;
}

.container{
    padding:20px;
}

.products{
    display:grid;
    grid-template-columns:repeat(auto-fit,minmax(200px,1fr));
    gap:20px;
}

.product{
    background:white;
    padding:15px;
    border-radius:10px;
    box-shadow:0 2px 10px rgba(0,0,0,0.1);
    text-align:center;
}

.product img{
    width:100%;
    height:150px;
    object-fit:cover;
    border-radius:10px;
}

button{
    background:#0a7c2f;
    color:white;
    border:none;
    padding:8px 12px;
    border-radius:5px;
    cursor:pointer;
}

.cart{
    margin-top:30px;
    background:white;
    padding:20px;
    border-radius:10px;
}

input,select{
    width:100%;
    padding:8px;
    margin:5px 0;
}

.hidden{
    display:none;
}

.admin-panel{
    max-width:400px;
    margin:auto;
    background:white;
    padding:20px;
    border-radius:10px;
    box-shadow:0 2px 10px rgba(0,0,0,0.1);
}
</style>
</head>

<body>

<header>
<h1>C Mart</h1>
<button onclick="openAdmin()">Owner Panel</button>
</header>

<div class="container">

<h2>Products</h2>
<div class="products" id="productList"></div>

<div class="cart">
<h3>Cart</h3>
<div id="cartItems"></div>
<h4>Total: ₹<span id="total">0</span></h4>

<h3>Checkout</h3>
<input type="text" id="customerName" placeholder="Full Name">
<input type="text" id="customerAddress" placeholder="Delivery Address">
<input type="email" id="customerEmail" placeholder="Email">

<select id="paymentMethod">
<option value="COD">Cash on Delivery</option>
<option value="Online">Online Payment</option>
</select>

<button onclick="placeOrder()">Place Order</button>
</div>

<div id="adminSection" class="hidden">
<div class="admin-panel">
<h2>Add Product</h2>
<input type="text" id="pname" placeholder="Product Name">
<input type="number" id="pprice" placeholder="Price">
<input type="text" id="pimage" placeholder="Image URL">
<button onclick="addProduct()">Add Product</button>
</div>
</div>

</div>

<script>
/* ---------------- CONFIG ---------------- */

// Replace these
const ADMIN_PIN = "1234"; 
const RAZORPAY_KEY = "YOUR_RAZORPAY_KEY";
emailjs.init("YOUR_EMAILJS_PUBLIC_KEY");

/* --------------------------------------- */

let products = JSON.parse(localStorage.getItem("products")) || [
{ name:"Rice 1kg", price:50, image:"https://via.placeholder.com/200" },
{ name:"Milk 1L", price:30, image:"https://via.placeholder.com/200" }
];

let cart = [];

function saveProducts(){
localStorage.setItem("products",JSON.stringify(products));
}

function renderProducts(){
const list=document.getElementById("productList");
list.innerHTML="";
products.forEach((p,index)=>{
list.innerHTML+=`
<div class="product">
<img src="${p.image}">
<h4>${p.name}</h4>
<p>₹${p.price}</p>
<button onclick="addToCart(${index})">Add to Cart</button>
</div>`;
});
}

function addToCart(i){
cart.push(products[i]);
renderCart();
}

function renderCart(){
let items=document.getElementById("cartItems");
items.innerHTML="";
let total=0;
cart.forEach(item=>{
total+=item.price;
items.innerHTML+=`<p>${item.name} - ₹${item.price}</p>`;
});
document.getElementById("total").innerText=total;
}

function placeOrder(){

if(cart.length==0){
alert("Cart is empty!");
return;
}

let name=document.getElementById("customerName").value;
let address=document.getElementById("customerAddress").value;
let email=document.getElementById("customerEmail").value;
let payment=document.getElementById("paymentMethod").value;
let total=cart.reduce((sum,item)=>sum+item.price,0);

if(!name || !address || !email){
alert("Please fill all details");
return;
}

if(payment==="COD"){
sendEmail(name,address,email,payment,total);
alert("Order placed successfully (COD)");
cart=[];
renderCart();
}
else{
var options = {
"key": RAZORPAY_KEY,
"amount": total*100,
"currency": "INR",
"name": "C Mart",
"description": "Grocery Purchase",
"handler": function(response){
sendEmail(name,address,email,"Online Payment",total);
alert("Payment successful! Order placed.");
cart=[];
renderCart();
}
};
var rzp = new Razorpay(options);
rzp.open();
}

}

function sendEmail(name,address,email,payment,total){
let orderItems=cart.map(item=>item.name).join(", ");

emailjs.send("YOUR_SERVICE_ID","YOUR_TEMPLATE_ID",{
customer_name:name,
customer_email:email,
customer_address:address,
order_items:orderItems,
payment_method:payment,
total_amount:total
});
}

/* -------- OWNER PANEL -------- */

function openAdmin(){
let pin=prompt("Enter Owner PIN:");
if(pin===ADMIN_PIN){
document.getElementById("adminSection").classList.toggle("hidden");
}else{
alert("Access Denied");
}
}

function addProduct(){
let name=document.getElementById("pname").value;
let price=document.getElementById("pprice").value;
let image=document.getElementById("pimage").value;

if(!name || !price || !image){
alert("Fill all fields");
return;
}

products.push({name,price:Number(price),image});
saveProducts();
renderProducts();
alert("Product added!");
}

renderProducts();
</script>

</body>
</html>
