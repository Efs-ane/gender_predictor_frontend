const API_URL = "https://demografix-c63625879fc2.herokuapp.com/api/classify/";

const form = document.getElementById("predict-form");
const nameInput = document.getElementById("name-input");
const statusMessage = document.getElementById("status-message");
const resultCard = document.getElementById("result-card");
const resultName = document.getElementById("result-name");
const resultGender = document.getElementById("result-gender");
const resultProbability = document.getElementById("result-probability");
const resultSampleSize = document.getElementById("result-sample-size");
const resultConfidence = document.getElementById("result-confidence");
const confidencePill = document.getElementById("confidence-pill");

function setStatus(message, type = "") {
  statusMessage.textContent = message;
  statusMessage.className = "status-message";

  if (type) {
    statusMessage.classList.add(type);
  }
}

function setResultVisible(isVisible) {
  resultCard.classList.toggle("hidden", !isVisible);
}

function formatGender(gender) {
  if (gender === null || gender === undefined || gender === "null") {
    return "Unknown";
  }

  return gender.charAt(0).toUpperCase() + gender.slice(1);
}

function formatProbability(value) {
  if (value === null || value === undefined) return "-";
  return `${Number(value).toFixed(2)}`;
}

function updateResult(data) {
  const { name, gender, probability, sample_size, is_confident } = data;

  resultName.textContent = name || "-";
  resultGender.textContent = formatGender(gender);
  resultProbability.textContent = `${formatProbability(probability)} / 1.00`;
  resultSampleSize.textContent = sample_size ?? "-";
  resultConfidence.textContent = is_confident ? "High" : "Low";

  confidencePill.textContent = is_confident ? "Confident" : "Less confident";
  confidencePill.style.background = is_confident
    ? "rgba(52, 211, 153, 0.12)"
    : "rgba(251, 191, 36, 0.12)";
  confidencePill.style.color = is_confident ? "#bbf7d0" : "#fde68a";
  confidencePill.style.borderColor = is_confident
    ? "rgba(52, 211, 153, 0.3)"
    : "rgba(251, 191, 36, 0.35)";

  setResultVisible(true);
}

async function predictGender(name) {
  const trimmedName = name.trim();

  if (!trimmedName) {
    setStatus("Please enter a first name.", "error");
    setResultVisible(false);
    return;
  }

  if (!/^[A-Za-z]+$/.test(trimmedName)) {
    setStatus("Name must contain only letters (A-Z or a-z).", "error");
    setResultVisible(false);
    return;
  }

  setStatus("Predicting gender...", "");

  try {
    const response = await fetch(`${API_URL}?name=${encodeURIComponent(trimmedName)}`);

    if (!response.ok) {
      const errorData = await response.json().catch(() => null);
      const message =
        errorData && errorData.message
          ? errorData.message
          : "Something went wrong while predicting the name.";

      throw new Error(message);
    }

    const data = await response.json();
    updateResult(data);
    setStatus("Prediction complete.", "success");
  } catch (error) {
    setStatus(error.message || "Unable to fetch prediction.", "error");
    setResultVisible(false);
  }
}

form.addEventListener("submit", (event) => {
  event.preventDefault();
  predictGender(nameInput.value);
});

nameInput.addEventListener("keydown", (event) => {
  if (event.key === "Enter") {
    event.preventDefault();
    predictGender(nameInput.value);
  }
});

setStatus("Try a name like sam, alice, or james.");
