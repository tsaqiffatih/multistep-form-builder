<template>
  <div class="flex items-center justify-center min-h-screen">
    <div
      class="max-w-lg w-full bg-white p-6 md:p-8 shadow-lg border border-gray-200 rounded-lg"
    >
      <div v-if="currentStep <= formConfig.length">
        <h2 class="text-xl md:text-2xl font-bold mb-2 text-center">
          {{ currentForm.title }}
        </h2>
        <p class="text-gray-600 mb-4 text-center">
          {{ currentForm.description }}
        </p>

        <form @submit.prevent="handleSubmit" class="space-y-4">
          <div
            v-for="(field, index) in currentForm.fields"
            :key="index"
            class="w-full mb-4"
          >
            <label class="block font-medium mb-2">{{ field.label }}</label>
            <input
              v-if="field.type === 'textfield'"
              type="text"
              :placeholder="field.placeholder"
              v-model="formData[field.label]"
              class="w-full px-3 py-2 border rounded"
            />
            <textarea
              v-if="field.type === 'textarea'"
              :placeholder="field.placeholder"
              v-model="formData[field.label]"
              class="w-full px-3 py-2 border rounded"
            ></textarea>
            <div v-if="field.type === 'radio'" class="flex gap-2">
              <label
                v-for="option in field.options"
                :key="isObject(option) ? option.value : option"
              >
                <input
                  type="radio"
                  :value="isObject(option) ? option.value : option"
                  v-model="formData[field.label]"
                  class="mr-2"
                />
                {{ isObject(option) ? option.label : option }}
              </label>
            </div>

            <select
              v-if="field.type === 'autocomplete'"
              v-model="formData[field.label]"
              class="w-full px-3 py-2 border rounded"
            >
              <option
                v-for="option in field.options"
                :key="isObject(option) ? option.value : option"
                :value="isObject(option) ? option.value : option"
              >
                {{ isObject(option) ? option.label : option }}
              </option>
            </select>
            <p v-if="errors[field.label]" class="text-red-500 text-sm">
              {{ errors[field.label] }}
            </p>
          </div>

          <div class="flex flex-col md:flex-row justify-between gap-4 mt-4">
            <!-- Previous Button -->
            <button
              v-if="currentStep > 1"
              type="button"
              @click="prevStep"
              class="px-4 py-2 bg-gray-300 rounded w-full md:w-auto"
            >
              Previous
            </button>
            <!-- Previous Button -->

            <!-- Next Button -->
            <button
              v-if="currentStep < formConfig.length"
              type="button"
              @click="validateAndNext"
              class="px-4 py-2 bg-blue-500 text-white rounded w-full md:w-auto"
            >
              Next
            </button>
            <!-- Next Button -->

            <!--Submit Button-->
            <button
              v-if="currentStep === formConfig.length"
              type="submit"
              class="px-4 py-2 bg-green-500 text-white rounded w-full md:w-auto"
            >
              Submit
            </button>
            <!--Submit Button-->
          </div>
        </form>
      </div>
    </div>
  </div>
</template>
  
  <script lang="ts">
import { defineComponent, ref, computed } from "vue";
import { formConfig } from "../data/formConfig";
import Swal from "sweetalert2";

export default defineComponent({
  setup() {
    const currentStep = ref(1);
    const formData = ref<{ [key: string]: any }>({});
    const errors = ref<{ [key: string]: string }>({});
    const config = formConfig;

    const currentForm = computed(() => config[currentStep.value - 1]);

    const isObject = (item: any): item is { label: string; value: string } => {
      return typeof item === "object" && item !== null && "label" in item && "value" in item;
    };

    const nextStep = () => {
      if (currentStep.value < config.length) {
        currentStep.value++;
        errors.value = {};
      }
    };

    const prevStep = () => {
      if (currentStep.value > 1) {
        currentStep.value--;
      }
    };

    const validateFields = () => {
      errors.value = {};
      let isValid = true;

      currentForm.value.fields.forEach((field) => {
        if (field.required && !formData.value[field.label]) {
          errors.value[field.label] = `${field.label} is required`;
          isValid = false;
        }
      });

      return isValid;
    };

    const validateAndNext = () => {
      if (validateFields()) {
        nextStep();
      }
    };

    const handleSubmit = () => {
      if (validateFields()) {
        Swal.fire(
          "Form Submitted",
          "Your form has been successfully submitted!!",
          "success"
        ).then(() => {
          window.location.reload();
        });
        console.log(formData.value);

        // setTimeout(() => {
        //   window.location.reload();
        // }, 2000);
      }
    };

    return {
      currentStep,
      currentForm,
      formData,
      errors,
      nextStep,
      prevStep,
      validateFields,
      validateAndNext,
      handleSubmit,
      formConfig: config,
      isObject
    };
  },
});
</script>
  
  <style scoped>
.text-red-500 {
  color: #f56565;
}
</style>
  