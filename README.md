<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Red Trabajo</title>

<script src="https://cdn.tailwindcss.com"></script>

</head>

<body class="bg-gray-100 p-6">

<h1 class="text-3xl font-bold mb-6 text-center">Red Trabajo</h1>

<div id="authSection" class="max-w-md mx-auto bg-white p-4 rounded shadow">

<h2 class="text-xl mb-4">Login / Sign Up</h2>

<input id="email" class="border p-2 w-full mb-2" placeholder="Email">
<input id="password" type="password" class="border p-2 w-full mb-2" placeholder="Password">

<select id="role" class="border p-2 w-full mb-3">
<option value="worker">Worker</option>
<option value="employer">Employer</option>
</select>

<button onclick="signup()" class="bg-green-500 text-white px-4 py-2 rounded w-full mb-2">Sign Up</button>
<button onclick="login()" class="bg-blue-500 text-white px-4 py-2 rounded w-full">Login</button>

</div>

<div id="appSection" class="hidden max-w-xl mx-auto">

<div class="flex justify-between mb-4">
<h2 class="text-xl">Dashboard</h2>
<button onclick="logout()" class="bg-red-500 text-white px-3 py-1 rounded">Logout</button>
</div>

<div id="employerPanel" class="hidden bg-white p-4 rounded shadow mb-4">

<h3 class="font-bold mb-2">Post Job</h3>

<input id="title" class="border p-2 w-full mb-2" placeholder="Job Title">
<input id="description" class="border p-2 w-full mb-2" placeholder="Description">
<input id="phone" class="border p-2 w-full mb-2" placeholder="Phone">

<button onclick="postJob()" class="bg-green-500 text-white px-4 py-2 rounded">Post Job</button>

</div>

<div class="bg-white p-4 rounded shadow">

<h3 class="font-bold mb-3">Available Jobs</h3>

<div id="jobs"></div>

</div>

</div>

<script type="module">

import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";

import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, onAuthStateChanged }

from "https://www.gstatic.com/firebasejs/10.12.0/firebase-auth.js";

import { getFirestore, doc, setDoc, getDoc, addDoc, collection, getDocs }

from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

const firebaseConfig = {

apiKey: "YOUR_API_KEY",
authDomain: "YOUR_DOMAIN",
projectId: "YOUR_PROJECT_ID",
storageBucket: "YOUR_BUCKET",
messagingSenderId: "YOUR_ID",
appId: "YOUR_APP_ID"

};

const app = initializeApp(firebaseConfig);

const auth = getAuth(app);
const db = getFirestore(app);

window.signup = async function(){

const email = document.getElementById("email").value;
const password = document.getElementById("password").value;
const role = document.getElementById("role").value;

const userCredential = await createUserWithEmailAndPassword(auth,email,password);

await setDoc(doc(db,"users",userCredential.user.uid),{

role: role,
createdAt: Date.now()

});

alert("Account created");

}

window.login = async function(){

const email = document.getElementById("email").value;
const password = document.getElementById("password").value;

await signInWithEmailAndPassword(auth,email,password);

}

window.logout = function(){

signOut(auth);

}

onAuthStateChanged(auth, async (user)=>{

if(user){

document.getElementById("authSection").classList.add("hidden");
document.getElementById("appSection").classList.remove("hidden");

const userDoc = await getDoc(doc(db,"users",user.uid));
const role = userDoc.data().role;

if(role === "employer"){

document.getElementById("employerPanel").classList.remove("hidden");

}

loadJobs();

}else{

document.getElementById("authSection").classList.remove("hidden");
document.getElementById("appSection").classList.add("hidden");

}

});

window.postJob = async function(){

navigator.geolocation.getCurrentPosition(async(position)=>{

const lat = position.coords.latitude;
const lng = position.coords.longitude;

const title = document.getElementById("title").value;
const description = document.getElementById("description").value;
const phone = document.getElementById("phone").value;

await addDoc(collection(db,"jobs"),{

title,
description,
phone,
createdAt: Date.now(),
location:{lat,lng}

});

alert("Job posted");

loadJobs();

});

}

async function loadJobs(){

const jobsDiv = document.getElementById("jobs");
jobsDiv.innerHTML="";

const snapshot = await getDocs(collection(db,"jobs"));

snapshot.forEach((docu)=>{

const job = docu.data();

const now = Date.now();

if(now - job.createdAt < 604800000){

const card = document.createElement("div");

card.className="border p-3 mb-2";

card.innerHTML = `
<h4 class="font-bold">${job.title}</h4>
<p>${job.description}</p>
<a class="text-blue-500" href="tel:${job.phone}">Call Employer</a>
`;

jobsDiv.appendChild(card);

}

});

}

</script>

</body>
</html># red-trabajo
