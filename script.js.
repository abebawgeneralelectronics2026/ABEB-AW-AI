
"use strict";

// ========================================
// ABEB AW AI - API CONNECTIONS
// ========================================

const apiURL =
  "https://old-rain-2f82abeb-aw-ai-api.jacksonabebe98.workers.dev/";

const speakURL =
  "https://abeb-aw-ai-brain.jacksonabebe98.workers.dev/speak";


// ========================================
// HTML ELEMENTS
// ========================================

const aiMessage =
  document.getElementById("ai-message");

const aiResponse =
  document.getElementById("ai-response");

const aiSend =
  document.getElementById("ai-send");

const aiListen =
  document.getElementById("ai-listen");

const aiLanguage =
  document.getElementById("ai-language");

const aiSpeak =
  document.getElementById("ai-speak");

const aiStatus =
  document.getElementById("ai-status");

const connectionStatus =
  document.getElementById("connection-status");


// ========================================
// VARIABLES
// ========================================

let aiRecognition = null;

let aiListening = false;

let lastAIAnswer = "";

let currentAudio = null;

let currentAudioURL = null;


// ========================================
// STOP AUDIO
// ========================================

function stopAIAudio() {

  if (currentAudio) {

    currentAudio.pause();

    currentAudio.currentTime = 0;

    currentAudio.onended = null;

    currentAudio.onerror = null;

    currentAudio = null;

  }

  if (currentAudioURL) {

    URL.revokeObjectURL(currentAudioURL);

    currentAudioURL = null;

  }

  aiSpeak.textContent = "🔊 SPEAK RESPONSE";

  aiSpeak.disabled = !lastAIAnswer;

}


// ========================================
// CONVERT RAW PCM AUDIO TO WAV
// ========================================

function pcmToWav(pcmBytes, sampleRate) {

  const headerSize = 44;

  const wavBuffer = new ArrayBuffer(
    headerSize + pcmBytes.length
  );

  const view = new DataView(wavBuffer);

  function writeString(offset, value) {

    for (let i = 0; i < value.length; i++) {

      view.setUint8(
        offset + i,
        value.charCodeAt(i)
      );

    }

  }

  writeString(0, "RIFF");

  view.setUint32(
    4,
    36 + pcmBytes.length,
    true
  );

  writeString(8, "WAVE");

  writeString(12, "fmt ");

  view.setUint32(16, 16, true);

  view.setUint16(20, 1, true);

  view.setUint16(22, 1, true);

  view.setUint32(24, sampleRate, true);

  view.setUint32(28, sampleRate * 2, true);

  view.setUint16(32, 2, true);

  view.setUint16(34, 16, true);

  writeString(36, "data");

  view.setUint32(
    40,
    pcmBytes.length,
    true
  );

  new Uint8Array(wavBuffer, headerSize)
    .set(pcmBytes);

  return wavBuffer;

}


// ========================================
// MICROPHONE INPUT
// ========================================

aiListen.addEventListener("click", function() {

  const SpeechRecognition =
    window.SpeechRecognition ||
    window.webkitSpeechRecognition;


  if (!SpeechRecognition) {

    aiStatus.textContent =
      "Speech recognition is unavailable. Please use Google Chrome.";

    return;

  }


  if (aiListening) {

    aiRecognition.stop();

    return;

  }


  aiRecognition = new SpeechRecognition();


  const selectedLanguage = aiLanguage.value;


  aiRecognition.lang =
    selectedLanguage === "auto"
      ? (navigator.language || "en-US")
      : selectedLanguage;


  aiRecognition.continuous = false;

  aiRecognition.interimResults = true;

  aiRecognition.maxAlternatives = 1;


  aiRecognition.onstart = function() {

    aiListening = true;

    aiListen.textContent = "⏹ STOP";

    aiStatus.textContent = "Listening... Speak now.";

  };


  aiRecognition.onresult = function(event) {

    let spokenText = "";

    for (
      let i = 0;
      i < event.results.length;
      i++
    ) {

      spokenText +=
        event.results[i][0].transcript;

    }

    aiMessage.value = spokenText;

    aiResponse.textContent =
      "I heard: " + spokenText;

  };


  aiRecognition.onerror = function(event) {

    aiStatus.textContent =
      "Microphone problem: " + event.error;

  };


  aiRecognition.onend = function() {

    aiListening = false;

    aiListen.textContent = "🎤 SPEAK";

  };


  try {

    aiRecognition.start();

  } catch (error) {

    aiStatus.textContent = error.message;

    aiListening = false;

    aiListen.textContent = "🎤 SPEAK";

  }

});


// ========================================
// SEND MESSAGE TO ABEB AW AI
// ========================================

aiSend.addEventListener("click", async function() {

  const message = aiMessage.value.trim();


  if (!message) {

    aiStatus.textContent =
      "Please write or speak a message first.";

    return;

  }


  stopAIAudio();

  lastAIAnswer = "";

  aiSpeak.disabled = true;

  aiSend.disabled = true;

  aiResponse.textContent =
    "ABEB AW AI is thinking...";

  aiStatus.textContent =
    "Sending your message...";


  try {

    const response = await fetch(apiURL, {

      method: "POST",

      headers: {
        "Content-Type": "application/json"
      },

      body: JSON.stringify({

        message: message,

        language: aiLanguage.value

      })

    });


    const data = await response.json();


    if (!response.ok) {

      throw new Error(
        data.error || "Connection failed."
      );

    }


    const answer =

      data.output_text ||

      data.reply ||

      data.response ||

      data.output?.flatMap(item => item.content || [])

        .filter(item => item.type === "output_text")

        .map(item => item.text)

        .join("\n");


    lastAIAnswer = answer || "";

    aiResponse.textContent =

      lastAIAnswer ||

      "The AI returned no text response.";


    aiSpeak.disabled = !lastAIAnswer;

    aiSpeak.textContent = "🔊 SPEAK RESPONSE";

    aiStatus.textContent =
      "Message received successfully.";


  } catch (error) {

    aiResponse.textContent =
      "Connection error: " + error.message;

    aiStatus.textContent =
      "Please check your internet connection.";

  } finally {

    aiSend.disabled = false;

  }

});


// ========================================
// SPEAK AI RESPONSE
// USER MUST TAP THE BUTTON
// ========================================

aiSpeak.addEventListener("click", async function() {


  // STOP CURRENT AUDIO IF PLAYING

  if (currentAudio && !currentAudio.paused) {

    stopAIAudio();

    aiStatus.textContent = "Speech stopped.";

    return;

  }


  if (!lastAIAnswer) {

    return;

  }


  stopAIAudio();

  aiSpeak.disabled = true;

  aiSpeak.textContent =
    "🔊 GENERATING VOICE...";


  try {

    const response = await fetch(speakURL, {

      method: "POST",

      headers: {

        "Content-Type": "application/json"

      },

      body: JSON.stringify({

        text: lastAIAnswer

      })

    });


    const data = await response.json();


    if (!response.ok) {

      throw new Error(
        data.error || "Voice generation failed."
      );

    }


    if (!data.audio) {

      throw new Error(
        "No audio received from ABEB AW AI."
      );

    }


    // DECODE BASE64 AUDIO

    const binary = atob(data.audio);

    const bytes = new Uint8Array(binary.length);

    for (let i = 0; i < binary.length; i++) {

      bytes[i] = binary.charCodeAt(i);

    }


    const mimeType =
      data.mimeType || "audio/wav";

    let audioBlob;


    // HANDLE RAW PCM AUDIO

    if (/audio\/l16|audio\/pcm/i.test(mimeType)) {

      const rateMatch =
        mimeType.match(/rate=(\d+)/i);

      const sampleRate = rateMatch
        ? Number(rateMatch[1])
        : 24000;


      const wavBuffer = pcmToWav(
        bytes,
        sampleRate
      );


      audioBlob = new Blob(

        [wavBuffer],

        { type: "audio/wav" }

      );

    } else {

      // HANDLE WAV OR OTHER AUDIO FORMATS

      audioBlob = new Blob(

        [bytes],

        { type: mimeType }

      );

    }


    currentAudioURL =
      URL.createObjectURL(audioBlob);


    currentAudio = new Audio(currentAudioURL);


    currentAudio.onended = function() {

      stopAIAudio();

      aiStatus.textContent =
        "Speech completed.";

    };


    currentAudio.onerror = function() {

      stopAIAudio();

      aiStatus.textContent =
        "Audio playback failed.";

    };


    await currentAudio.play();


    aiSpeak.textContent =
      "⏹ STOP SPEAKING";

    aiSpeak.disabled = false;

    aiStatus.textContent =
      "ABEB AW AI is speaking...";


  } catch (error) {

    stopAIAudio();

    aiSpeak.textContent =
      "🔊 SPEAK RESPONSE";

    aiSpeak.disabled = false;

    aiStatus.textContent =
      "Voice error: " + error.message;

  }

});


// ========================================
// CHECK AI CONNECTION
// ========================================

async function checkConnection() {

  try {

    const response = await fetch(apiURL);

    if (!response.ok) {

      throw new Error("Connection failed");

    }

    connectionStatus.textContent =
      "ABEB AW AI is connected.";

  } catch (error) {

    connectionStatus.textContent =
      "Connection unavailable.";

  }

}


checkConnection();

