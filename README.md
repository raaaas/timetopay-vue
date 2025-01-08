# Vue.js Expiration Date Function

A simple Vue.js function that implements automatic content expiration based on a server-provided date. The content gradually fades out as it approaches its expiration date.

## Features
- 🕒 Automatic expiration date checking
- 🌅 Smooth fade-out transitions 

## Usage

Add this code to your Vue.js project:

```javascript
// Add these imports at the top of your <script setup> section
import { onMounted, onUnmounted, ref, watch } from 'vue';
import axios from 'axios';

// Add these refs
const bodyOpacity = ref(1);
const expirationDate = ref(null);
const isExpired = ref(false);

// Function to check expiration and update opacity
const checkExpirationAndUpdateOpacity = async () => {
  try {
    // Replace with your actual API endpoint
    const response = await axios.get('YOUR_API_ENDPOINT');
    const data = response.data; // Assuming format "0:2025-01-10"
    
    const [status, dateStr] = data.split(':');
    
    if (status === '0') {
      expirationDate.value = new Date(dateStr);
      
      // Calculate remaining time
      const now = new Date();
      const totalDuration = expirationDate.value - now;
      
      if (totalDuration <= 0) {
        // If expired, set opacity to 0
        bodyOpacity.value = 0;
        isExpired.value = true;
      } else {
        // Calculate opacity based on remaining time
        // Assuming we want to start fading 30 days before expiration
        const fadeStartDuration = 30 * 24 * 60 * 60 * 1000; // 30 days in milliseconds
        
        if (totalDuration <= fadeStartDuration) {
          bodyOpacity.value = totalDuration / fadeStartDuration;
        }
        
        // Schedule next check
        setTimeout(checkExpirationAndUpdateOpacity, 60 * 60 * 1000); // Check every hour
      }
    }
  } catch (error) {
    console.error('Error checking expiration:', error);
  }
};

// Add to mounted lifecycle hook
onMounted(() => {
  checkExpirationAndUpdateOpacity();
});

// Add watch to apply opacity
watch(bodyOpacity, (newOpacity) => {
  document.body.style.opacity = newOpacity;
});
```

Add this to your template:
```vue
<div class="dark" :style="{ opacity: bodyOpacity }">
  <!-- Your content here -->
</div>
```

Add this CSS:
```css
.dark {
  transition: opacity 1s ease-in-out;
}
```

## Requirements
- Vue 3
- Axios

## API Response Format
The API should return a string in the format: `"0:YYYY-MM-DD"`
- First value (0): Status code
- Second value: Expiration date

Example: `"0:2025-01-10"`
