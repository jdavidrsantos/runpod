<template>
  <div class="container">
    <!-- Case Study Information Section -->
    <section class="case-study-info">
      <h2>RunPod Serverless Endpoint Case Study</h2>
      <p>This project demonstrates the use of a serverless endpoint to generate images from text input using the RunPod platform. Below, you can input a prompt to generate a corresponding image.</p>
    </section>

    <!-- Deliverables Section -->
    <section class="deliverables">
      <h3>Deliverables</h3>
      <ul>
        <li><strong>Demonstrate your working endpoint:</strong> Below, you can input a prompt and generate an image as a demonstration of the working endpoint.</li>
        <li><strong>Share relevant code or configurations:</strong> This task is fully documented in a pdf sent to shibin.bhaskaran@outsourced.co, which includes the integration with RunPod's API to handle the image generation process.</li>
      </ul>
    </section>

    <!-- Image Generator Section -->
    <section class="image-generator">
      <h1>Image Generator using RunPod</h1>
      <input
          v-model="prompt"
          placeholder="Enter your prompt..."
          :disabled="loading"
          class="input-prompt"
      />
      <button @click="generateImage" :disabled="loading" class="generate-btn">
        {{ loading ? 'Generating...' : 'Generate Image' }}
      </button>

      <!-- Spinner for loading state -->
      <div v-if="loading" class="spinner-container">
        <div class="spinner"></div>
      </div>

      <p v-if="error" class="error">{{ error }}</p>
      <img v-if="imageData" :src="'data:image/png;base64,' + imageData" alt="Generated Image" class="generated-image" />
    </section>

    <!-- Footer Section -->
    <footer class="footer">
      <p>Developed by Jesus David Rodriguez</p>
      <p>Email: <a href="mailto:jdavidrsantos@gmail.com">jdavidrsantos@gmail.com</a></p>
    </footer>
  </div>
</template>


<script setup>
import { ref } from 'vue'

const prompt = ref('')
const imageData = ref(null)
const loading = ref(false)
const error = ref(null)

const token = process.env.VUE_APP_RUNPOD_TOKEN
const baseUrl = 'https://api.runpod.ai/v2/asnirb164pa5yy'

async function generateImage() {
  if (!prompt.value) return
  loading.value = true
  error.value = null
  imageData.value = null

  try {
    // Send prompt to RunPod API
    const postResponse = await fetch(`${baseUrl}/run`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`
      },
      body: JSON.stringify({
        input: { text: prompt.value }
      })
    })

    const rawPost = await postResponse.text()
    if (!postResponse.ok) throw new Error(`POST failed: ${rawPost}`)

    const { id } = JSON.parse(rawPost)

    // Poll for status
    let status = ''
    let output = null
    while (status !== 'COMPLETED') {
      await new Promise(r => setTimeout(r, 3000)) // Wait 3 seconds

      const statusResponse = await fetch(`${baseUrl}/status/${id}`, {
        headers: {
          Authorization: `Bearer ${token}`
        }
      })

      const rawStatus = await statusResponse.text()
      if (!statusResponse.ok) throw new Error(`GET failed: ${rawStatus}`)

      const data = JSON.parse(rawStatus)
      status = data.status
      output = data.output?.output
    }

    imageData.value = output
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.container {
  max-width: 700px;
  margin: auto;
  padding: 1rem;
  text-align: center;
  font-family: sans-serif;
}

.deliverables {
  margin-top: 2rem;
  text-align: left;
  padding: 1rem;
  background-color: #e8f5e9;
  border-radius: 8px;
}

.code-example {
  margin-top: 2rem;
  text-align: left;
  padding: 1rem;
  background-color: #f4f4f4;
  border-radius: 8px;
}

input {
  width: 100%;
  padding: 0.5rem;
  margin-bottom: 1rem;
}

button {
  padding: 0.5rem 1rem;
}

img {
  margin-top: 1rem;
  max-width: 100%;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
}

.error {
  color: red;
  margin-top: 1rem;
}

pre {
  background-color: #f4f4f4;
  padding: 1rem;
  border-radius: 5px;
  overflow-x: auto;
}

/* Spinner styles */
.spinner-container {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
