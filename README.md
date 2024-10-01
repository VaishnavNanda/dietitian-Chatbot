# Dietitian-Chatbot
## AIM:
     to create an chatbot we are not working justing sitting simply

## HTML:
<!DOCTYPE html>
<html lang="en" dir="ltr">
    <head>
        <meta charset="utf-8">
        <title> JavaScript ChatBot </title>
        <link rel="stylesheet" href="style.css">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <!-- Google Fonts Link for Icons -->
        <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@48,400,0,0" />
    </head>

    <body class="show-ChatBot">
        <button class="chatbot-toggler">
            <span class="material-symbols-outlined">mode_comment</span>
            <span class="material-symbols-outlined">close</span>
        </button>
        <div class="ChatBot">
            <header>
                <h2>ChatBot</h2>
                <span class="material-symbols-outlined">close</span>
            </header>
            <ul class="chatbox">
                <li class="chat incoming">
                    <span class="material-symbols-outlined">smart_toy</span>
                    <p>Hi there 👋🏻 <br> How can I help you today? </p>
                </li>
                <li class="chat outgoing">
                    <p>Lorem ipsum dolor sit amet consectetur. </p>
                </li>
            </ul>
            <div class="chat-input">
                <textarea placeholder="Type a message..." required></textarea>
                <span id="send-btn" class="material-symbols-outlined">send</span>
            </div>
        </div>
    </body>
</html>


## JAVA SCRIPT:
const chatInput = document.querySelector(".chat-input textarea");
const sendChatBtn = document.querySelector(".chat-input span");
const chatbox = document.querySelector(".chatbox");

let userMessage;
const API_KEY = localStorage.getItem('API_KEY') || (localStorage.setItem("API_KEY", prompt("API Key?")) || localStorage.getItem('API_KEY'));


//Array to keep track of convo history
let convoHistory = [
    { role: "system", content: "You are a nutrition assistant bot. If the user message contains a food name, respond by providing the shelf life of that food and what signs of spoilage that food has. If the user message is not a food, reply with an error like \"That's not a food, please give me a food name,\". Be concise. Only 1 sentence at a time. If the user asks follow up questions regarding the food item they asked, answer concisely and be appropriate to the context. Adjust the conversation accordingly. If the user asks for a recipe for the ingredient, make sure to reply appropriately and give recipes but be concise and give the recipies in bullet points formatted correctly.  Be concise and context-aware"}
];

const createChatLi = (message, className) => {
    // Creating a chat <li> element with passed message and className
    const chatLi = document.createElement("li");
    chatLi.classList.add("chat", className);
    let chatContent = className === "outgoing" ? `<p>${message}</p>` : `<span style="font-size: 30px;">🤖</span> <p>${message}</p>`;
    chatLi.innerHTML = chatContent;
    return chatLi;
}

const generateResponse = (incomingChatLI) => {
    const API_URL = "https://api.openai.com/v1/chat/completions";
    const messageElement = incomingChatLI.querySelector("p");

    //Including convo history in API request
    let messages = convoHistory.concat([{role: "user", content: userMessage }]);

    const requestOptions = {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "Authorization": `Bearer ${API_KEY}`
        },
        body: JSON.stringify({
            model: "gpt-4",
            messages: messages // Now includes system message from convoHistory 
        })
        
    };

    // Sending POST request to API, getting response
    fetch(API_URL, requestOptions).then(res => res.json()).then(data => {
        let botResponse = data.choices[0].message.content;
        messageElement.textContent = botResponse;

        //Updating convo history with new messages
        convoHistory.push({ role: "user", content: userMessage });
        convoHistory.push({ role: "assistant", content: botResponse });

})

.catch((error) => {
    messageElement.textContent = "Oops! Something went wrong. Please try again.";
})

.finally(() => {
    chatbox.scrollTo(0, chatbox.scrollHeight);
});
};

const handleChat = () => {
    userMessage = chatInput.value.trim();
    if(!userMessage) return;

    // Appending user's message to the chatbox
    chatbox.appendChild(createChatLi(userMessage, "outgoing"));
    chatbox.scrollTo(0, chatbox.scrollHeight);

    setTimeout(() => {
        // Displaying "Thinking..." message from chatbot while waiting for response
        const incomingChatLI = createChatLi("Thinking...", "incoming")
        chatbox.appendChild(incomingChatLI);
        chatbox.scrollTo(0, chatbox.scrollHeight);
        generateResponse(incomingChatLI);

    }, 500);
    chatInput.value = '';
};

chatInput.addEventListener("keypress", function(event){
    if (event.key === "Enter"){
        event.preventDefault();
        handleChat();
    }
});



sendChatBtn.addEventListener("click", handleChat);

# CSS:
/*Import Google Font Poppins */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600&display=swap');
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: "Poppins", sans-serif;
}
body {
    background: #e3f2fd;
}

.chatbot-toggler {
    position: fixed;
    right: 40px;
    bottom: 35px;
    height: 50px;
    width: 50px;
    color: #fff;
    border: none;
    display: flex;
    align-items: center;
    justify-content: center;
    outline: none;
    cursor: pointer;
    background: #52116b;
    border-radius: 50%;
}

.chatbot-toggler span {
    position: absolute;

}

.show-ChatBot .chatbot-toggler span:first-child,
.chatbot-toggler span:last-child{
    opacity: 0;
}

.show-ChatBot .chatbot-toggler span:last-child {
    opacity: 1;
}

.ChatBot {
    position: fixed;
    right: 40px;
    bottom: 100px;
    width: 420px;
    transform: scale(0.5);
    opacity: 0;
    pointer-events: none;
    overflow: hidden;
    background: #fff;
    border-radius: 15px;
    box-shadow: 0 0 128px 0 rgba(0, 0, 0, 0.1),
                0 32px 64px -48px rgba(0, 0, 0, 0.5);
}

.show-ChatBot .ChatBot {
    transform: scale(1);
    opacity: 1;
    pointer-events: auto;
}

.ChatBot header {
    background: #558555;
    padding: 16px 0;
    text-align: center;
    position: relative;
}

.ChatBot header h2 {
    color: #fff;
    font-size: 1.4rem;
}

.ChatBot header span {
    position: absolute;
    right: 20px;
    top: 50%;
    color: #fff;
    cursor: pointer;
    display: none;
    transform: translateY(-50%);
}

.ChatBot .chatbox {
    height: 510px;
    overflow-y: auto;
    padding: 30px 20px 100px;
}

.chatbox .chat {
    display: flex;
}

.chatbox .incoming span {
    height: 32px;
    width: 32px;
    color: #fff;
    align-self: flex-end;
    text-align: center;
    line-height: 32px;
    border-radius: 4px;
    margin: 0 10px 7px 0;
}

.chatbox .outgoing {
    margin: 20px 0;
    justify-content: flex-end;
}

.chatbox .chat p {
    color: #fff;
    max-width: 75%;
    white-space: pre-wrap;
    font-size: 0.95rem;
    padding: 12px 16px;
    border-radius: 10px 10px 0 10px;
    background: #558555;
}

.chatbox .incoming p {
    color: #000;
    background: #f2f2f2;
    border-radius: 10px 10px 10px 0;
}

.ChatBot .chat-input {
    position: absolute;
    bottom: 0;
    width: 100%;
    display: flex;
    gap: 5px;
    background: #fff;
    padding: 5px 20px;
    border-top: 1px solid #ccc;
}

.chat-input textarea {
    height: 55px;
    width: 100%;
    border: none;
    outline: none;
    font-size: 0.95rem;
    resize: none;
    padding: 16px 15px 16px 0;
}

.chat-input span {
    align-self: flex-end;
    height: 55px;
    line-height: 55px;
    color: #52116b;
    font-size: 1.35rem;
    cursor: pointer;
    visibility: hidden;
}

.chat-input textarea:valid ~ span {
    visibility: visible;
}

@media(max-width: 490px) {
    .ChatBot {
        right: 0;
        bottom: 0;
        width: 100%;
        height: 100%;
        border-radius: 0;
    }
    .ChatBot .chatbox {
        height: 90%;
    }
    .ChatBot header span {
        display: block;
    }
}
