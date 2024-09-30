<template>
  <form v-if="!formSubmitted" @submit.prevent="submitForm">
    <!-- Description field -->
    <div>
      <label for="description">Description</label>
      <textarea
          id="description"
          v-model="form.description"
          :maxlength="255"
          required
          @input="validateDescription"
      ></textarea>
      <p>Characters left: {{ 255 - (form.description?.length ?? 0) }}</p>
      <p v-if="errors.description" class="error">{{ errors.description }}</p>
    </div>

    <!-- Send confirmation field -->
    <div class="radio-group">
      <label>Send confirmation</label>
      <label for="yes">
        <input id="yes" v-model="form.send_confirmation" :value="true" required type="radio">
        Yes
      </label>
      <label for="no">
        <input id="no" v-model="form.send_confirmation" :value="false" type="radio">
        No
      </label>
      <p v-if="errors.sendConfirmation" class="error">{{ errors.sendConfirmation }}</p>
    </div>

    <!-- VAT field -->
    <div>
      <label for="vat">VAT</label>
      <select id="vat" v-model="form.vat_percentage" required>
        <option :value="null" disabled selected>Choose VAT</option>
        <option :value="19">19%</option>
        <option :value="21">21%</option>
        <option :value="23">23%</option>
        <option :value="25">25%</option>
      </select>
      <p v-if="errors.vat" class="error">{{ errors.vat }}</p>
    </div>

    <!-- Price netto field -->
    <div>
      <label for="priceNetto">Price netto EUR</label>
      <input
          id="priceNetto"
          v-model.number="form.price_netto"
          :disabled="form.vat_percentage === null"
          min="0"
          required
          type="number"
          @input="validatePriceNetto"
      >
      <p v-if="errors.priceNetto" class="error">{{ errors.priceNetto }}</p>
    </div>

    <!-- Price brutto field -->
    <div>
      <label for="priceBrutto">Price brutto EUR</label>
      <input id="priceBrutto" :value="form.price_brutto" disabled type="text">
    </div>

    <button type="submit">Submit</button>
  </form>

  <div v-else class="success-message">
    <p>Congratulations! Your form has been submitted successfully.</p>
  </div>
</template>
<script lang="ts" setup>
import {computed, ref} from "vue";
import axios from 'axios';

const form = ref({
  description: null as string | null,
  send_confirmation: false,
  vat_percentage: null as number | null,
  price_netto: null as number | null,
  price_brutto: computed<number | null>((): number | null => {
    if (form.value.vat_percentage === null || form.value.price_netto === null) {
      return null
    }
    return Math.round(
        (form.value.price_netto * (1 + form.value.vat_percentage / 100) + Number.EPSILON) * 100
    ) / 100
  })
})

const errors = ref({
  description: null as string | null,
  sendConfirmation: null as string | null,
  vat: null as string | null,
  priceNetto: null as string | null
})

const formSubmitted = ref(false)
const loading = ref(false)

function validateDescription() {
  if (form.value.description === null)
    errors.value.description = "Text is required"
  else if (form.value.description.length > 255)
    errors.value.description = "You can’t enter more than 255 characters"
  else
    errors.value.description = null
}

function validatePriceNetto() {
  if (form.value.price_netto === null)
    errors.value.priceNetto = "Price is required"
  else
    errors.value.priceNetto = null
}

function validateVatPercentage() {
  if (form.value.vat_percentage === null)
    errors.value.vat = "VAT is required";

  else
    errors.value.vat = null;
}

// validate all fields
function validateForm(): boolean {
  validateDescription()
  validateVatPercentage();
  validatePriceNetto()
  return !Object.values(errors.value).some((error) => error !== null)
}

function submitForm() {
  // validate form before submitting
  if (!loading.value && validateForm()) {
    loading.value = true
    axios.post("https://httpbin.org/post", form.value)
        .then(() => {
          formSubmitted.value = true
        })
        .catch(() => {
          alert("Error")
        })
        .finally(() => {
          loading.value = false
        })
  }
}
</script>
<style scoped>
form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  max-width: 400px;
  margin: 0 auto;
}

label {
  font-weight: bold;
}

textarea, input:not([type="radio"]), select {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 0.25rem;
}

input[type="radio"] {
  margin-right: 0.5rem;
}

.radio-group {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.error {
  color: red;
}

.success-message {
  background: lightgreen;
  color: black;
  text-align: center;
  font-size: 1.5rem;
  margin-top: 2rem;
}
</style>