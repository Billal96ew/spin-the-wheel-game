let canvas = document.getElementById("wheelCanvas");
let ctx = canvas.getContext("2d");
let prizes = ["১০ টাকা", "২০ টাকা", "৫০ টাকা", "১০০ টাকা", "২০০ টাকা", "৫০০ টাকা", "১০০০ টাকা"];
let colors = ["red", "blue", "green", "orange", "purple", "pink", "yellow"];
let startAngle = 0;
let spinAngle = 0;
let spinning = false;
let selectedPrize = "";

// Adsterra রিডাইরেক্ট লিংক
const adsterraLink = "https://www.adsterra.com";

// কনফার্মেশন লিংক
const confirmationLink = "https://www.example.com";

// গেমটি কমপ্লিট না করলে বা বেক আসতে চাইলে Adsterra-র লিংকে রিডাইরেক্ট
window.addEventListener("beforeunload", function(event) {
    if (!document.getElementById("confirmationMessage").style.display === "block") {
        window.location.href = adsterraLink;
    }
});

function drawWheel() {
    let arc = Math.PI / (prizes.length / 2);
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    for (let i = 0; i < prizes.length; i++) {
        let angle = startAngle + i * arc;
        ctx.fillStyle = colors[i];
        ctx.beginPath();
        ctx.moveTo(150, 150);
        ctx.arc(150, 150, 150, angle, angle + arc);
        ctx.lineTo(150, 150);
        ctx.fill();
        ctx.fillStyle = "white";
        ctx.font = "14px Arial";
        ctx.fillText(prizes[i], 120 + 100 * Math.cos(angle + arc / 2), 150 + 100 * Math.sin(angle + arc / 2));
    }
}

function spinWheel() {
    if (spinning) return;
    spinning = true;
    let randomIndex = Math.floor(Math.random() * prizes.length);
    selectedPrize = prizes[randomIndex];
    spinAngle = 3600 + randomIndex * (360 / prizes.length);
    let spinInterval = setInterval(() => {
        startAngle += 10 * Math.PI / 180;
        if (spinAngle > 0) {
            spinAngle -= 10;
        } else {
            clearInterval(spinInterval);
            spinning = false;
            document.getElementById("rewardMessage").innerHTML = `<h2>🎉 অভিনন্দন! আপনি জিতেছেন ${selectedPrize}!</h2>`;
            document.getElementById("rewardMessage").style.display = "block";
            document.getElementById("withdrawSection").style.display = "block";
        }
        drawWheel();
    }, 20);
}

function showShareSection() {
    document.getElementById("withdrawSection").style.display = "none";
    document.getElementById("shareSection").style.display = "block";
    generateShareLink();
}

function generateShareLink() {
    let shareMessage = `আমি এই গেমটিতে ${selectedPrize} জিতেছি! আপনি চেষ্টা করে দেখুন: ${window.location.href}`;
    document.getElementById("shareLinkText").value = shareMessage;
}

function copyShareLink() {
    let shareLinkText = document.getElementById("shareLinkText");
    shareLinkText.select();
    document.execCommand("copy");
    let copyButton = document.querySelector(".copy-button");
    copyButton.textContent = "✔️ লিঙ্ক কপি হয়েছে!";
    copyButton.classList.add("copied");
    setTimeout(() => {
        copyButton.textContent = "📋 লিঙ্ক কপি করুন";
        copyButton.classList.remove("copied");
    }, 2000);
}

function shareOnMessenger() {
    let shareMessage = `আমি এই গেমটিতে ${selectedPrize} জিতেছি! আপনি চেষ্টা করে দেখুন: ${window.location.href}`;
    let messengerLink = `https://www.facebook.com/dialog/send?link=${encodeURIComponent(window.location.href)}&app_id=YOUR_APP_ID&redirect_uri=${encodeURIComponent(window.location.href)}`;
    window.open(messengerLink, "_blank");
}

function shareOnWhatsApp() {
    let shareMessage = `আমি এই গেমটিতে ${selectedPrize} জিতেছি! আপনি চেষ্টা করে দেখুন: ${window.location.href}`;
    let whatsappLink = `https://wa.me/?text=${encodeURIComponent(shareMessage)}`;
    window.open(whatsappLink, "_blank");
}

function showSubmitForm() {
    document.getElementById("shareSection").style.display = "none";
    document.getElementById("submitForm").style.display = "block";
    document.getElementById("amount").value = selectedPrize;
}

function validateForm() {
    let name = document.getElementById("name").value.trim();
    let phone = document.getElementById("phone").value.trim();
    let email = document.getElementById("email").value.trim();
    let isValid = true;

    if (name === "") {
        document.getElementById("nameError").style.display = "block";
        isValid = false;
    } else {
        document.getElementById("nameError").style.display = "none";
    }

    if (!/^\d{11}$/.test(phone)) {
        document.getElementById("phoneError").style.display = "block";
        isValid = false;
    } else {
        document.getElementById("phoneError").style.display = "none";
    }

    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
        document.getElementById("emailError").style.display = "block";
        isValid = false;
    } else {
        document.getElementById("emailError").style.display = "none";
    }

    return isValid;
}

document.getElementById("withdrawForm").addEventListener("submit", function(event) {
    event.preventDefault();
    if (!validateForm()) return;

    let name = document.getElementById("name").value;
    let phone = document.getElementById("phone").value;
    let email = document.getElementById("email").value;
    let amount = document.getElementById("amount").value;

    // সিমুলেটেড API কল
    fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "POST",
        body: JSON.stringify({ name, phone, email, amount }),
        headers: { "Content-type": "application/json; charset=UTF-8" }
    })
    .then(response => response.json())
    .then(data => {
        console.log("সার্ভার রেসপন্স:", data);
        document.getElementById("submitForm").style.display = "none";
        document.getElementById("loadingSection").style.display = "block";

        setTimeout(() => {
            document.getElementById("loadingSection").style.display = "none";
            document.getElementById("confirmationMessage").style.display = "block";
            document.getElementById("confirmationText").innerHTML = `
                আপনার উত্তোলনের অনুরোধ সফলভাবে জমা হয়েছে।<br>
                নাম: ${name}<br>
                ফোন নম্বর: ${phone}<br>
                ইমেইল: ${email}<br>
                উত্তোলনের পরিমাণ: ${amount}
            `;
        }, 3000);
    })
    .catch(error => console.error("ত্রুটি:", error));
});

function confirmAction() {
    window.location.href = confirmationLink;
}

function resetGame() {
    document.getElementById("confirmationMessage").style.display = "none";
    document.getElementById("rewardMessage").style.display = "none";
    document.getElementById("withdrawSection").style.display = "none";
    document.getElementById("submitForm").style.display = "none";
    document.getElementById("loadingSection").style.display = "none";
    document.getElementById("shareSection").style.display = "none";
    document.getElementById("name").value = "";
    document.getElementById("phone").value = "";
    document.getElementById("email").value = "";
    document.getElementById("amount").value = "";
    selectedPrize = "";
    drawWheel();
}

drawWheel();