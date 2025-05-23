<script setup>
import {reactive, watch} from 'vue';
import {z} from 'zod';
import {useI18n} from 'vue-i18n';
import {useNuxtApp} from "#app";
import SubmitMessagePopup from '~/components/SubmitMessagePopup.vue';


const {t} = useI18n();
const {$axios} = useNuxtApp();
const api = $axios()
const showPopup = ref(false);
const contactQuestions = [
  {
    label: t('contact.form.name'),
    type: "text",
    placeholder: t('contact.form.name_placeholder'),
    required: true,
    id: "name",
    icon: "user"
  },
  {
    label: t('contact.form.email'),
    type: "email",
    placeholder: t('contact.form.email_placeholder'),
    required: true,
    id: "email",
    icon: "mail"
  },
  {
    label: t('contact.form.phone'),
    type: "tel",
    placeholder: t('contact.form.phone_placeholder'),
    required: false,
    id: "contact_number",
    icon: "phone"
  },
  {
    label: t('contact.form.subject'),
    type: "text",
    placeholder: t('contact.form.subject_placeholder'),
    required: false,
    id: "subject",
    icon: "edit"
  },
  {
    label: t('contact.form.message'),
    type: "textarea",
    placeholder: t('contact.form.message_placeholder'),
    required: true,
    id: "message",
    icon: "message-circle"
  }
]
const formSchema = z.object({
  name: z
      .string()
      .min(8, 'Name must be at least 8 characters long'),

  email: z
      .string()
      .email('Invalid email format'),

  contact_number: z
      .string()
      .regex(/^\d{8,15}$/, 'Invalid phone number'),

  subject: z
      .string()
      .min(8, 'Subject name must be at least 8 characters long'),

  message: z
      .string()
      .min(20, 'Message must be at least 20 characters long')
});
const form = reactive({});
const errors = reactive({});

contactQuestions.forEach((question) => {
  form[question.id] = "";
  errors[question.id] = "";
});

function validateField(field) {
  try {
    formSchema.shape[field].parse(form[field]);
    errors[field] = "";
  } catch (error) {
    console.error(`Validation failed for field: ${field}`, error);
    errors[field] = error.errors ? error.errors[0].message : error.message;
  }
}

contactQuestions.forEach((question) => {
  watch(() => form[question.id], () => validateField(question.id));
});

async function handleSubmit() {
  form.Date = new Date().toLocaleDateString("en-GB");

  const validationResults = formSchema.safeParse(form);

  if (!validationResults.success) {
    console.log('Validation Errors:', validationResults.error.errors);
    alert("Please correct the errors in the form.");
    return;
  }

  try {
    const payload = {
      name: form.name,
      email: form.email,
      contact_number: form.contact_number,
      subject: form.subject,
      message: form.message,
      date: form.Date
    };

    const response = await api.post("/requests/contact_message/", payload);

    console.log("Form submitted successfully:", response.data);
    alert("Your message has been sent successfully!");
    Object.keys(form).forEach((key) => {
      form[key] = "";
    });

    setTimeout(() => {
      showPopup.value = true;

      setTimeout(() => {
        location.reload();
      }, 2000);

    }, 1000);

  } catch (error) {
    console.error("Error submitting form:", error);
    alert("There was an error while submitting the form. Please try again.");
  }
}


const contactImage = "/images/contact-image.jpg"

</script>

<template>
  <section class="contact-section">
    <div class="container">

      <div class="contact-info-container">

        <div class="contact-image-container">
          <img :src="contactImage" alt="contact-png" class="contact-image"/>
        </div>

        <div class="contact-info">
          <h3>{{ t('contact.title') }}</h3>
          <span>{{ t('contact.description') }}</span>
        </div>
      </div>

      <div class="contact-form">

        <h2>{{ t('contact.your_details') }}</h2>

        <form @submit.prevent="handleSubmit">
          <div class="contact-form">
            <div class="info" v-for="(question, index) in contactQuestions " :key="index">
              <label class="question-title" :for="question.label">{{ question.label }}</label>

              <input
                  v-if="question.type === 'text'||question.type === 'email' ||question.type === 'tel'"
                  :type="question.type"
                  v-model="form[question.id]"
                  :placeholder="question.placeholder"
                  :id="question.label"
                  @input="validateField(question.id)"
              />

              <span v-if="errors[question.id]" class="error">{{ errors[question.id] }}</span>

              <textarea
                  v-if="question.type === 'textarea'"
                  :id="question.label"
                  :name="question.label"
                  :placeholder="question.placeholder"
                  v-model="form[question.id]"
              />

            </div>
          </div>

          <div>
            <button @click.once="isPopupVisible = true" class="contact-submit" type="submit">
              {{ t('contact.send_message') }}
            </button>

            <SubmitMessagePopup :show="showPopup" @update:show="showPopup = $event">

            </SubmitMessagePopup>

          </div>

        </form>
      </div>

    </div>
  </section>
</template>

<style scoped>

.contact-section {
  background-color: var(--bg-color);
  width: 100%;
  margin: 0 auto;
  padding: 5rem 0;
}

.container {
  display: grid;
  grid-template-columns: 1fr 2fr;
  flex-wrap: wrap;
  max-width: 1000px;
  margin: 0 auto;
}

@media (max-width: 800px) {
  .contact-section {
    margin: 0.5rem;
  }

  .container {
    grid-template-columns: 1fr;
    max-width: 1200px;
  }
}

.contact-image-container .contact-image {
  width: 100%;
  max-width: 350px;
  margin: 0 auto;
  height: 100%;
  object-fit: cover;
  align-content: center;
}

.contact-info-container {
  display: block;
  align-items: center;
  text-align: center;
}

.contact-info-container div {
  text-align: center;
  align-items: center;
  margin: 3rem auto;
}

.contact-info-container .contact-info h3 {
  color: var(--secondary-color);
  font-size: 1.5rem;
}

.contact-info-container .contact-info span {
  color: var(--text-hover);
  font-size: 1rem;
}


.container .contact-form {
  flex: 1;
  padding: 0 2rem;
}

@media (max-width: 800px) {
  .container div {
    display: block;
  }

  .container .contact-form {
    flex: 1;
    padding: 1rem;
  }
}

@media (max-width: 1200px) {
  .container {
    display: block;
  }
}

.container .description h2 {
  font-size: 1.2rem;
  padding: .5rem 0;
  font-weight: bold;
  color: var(--primary-color);
  text-align: center;
}

.container .description p {
  font-size: 1rem;
  padding: 1rem 0;
  font-weight: normal;
  color: var(--primary-color);
  text-align: justify;
}

.contact-form > h2 {
  font-size: 1.5rem;
  color: var(--primary-color);
  text-align: center;
  padding: 1rem 0;
}

.contact-form {
  display: flex;
  flex-wrap: wrap;
  flex-direction: row;
  gap: 1rem;
  margin-top: 0.5rem !important;
}

.info {
  flex-basis: calc(50% - 10px);
  box-sizing: border-box;
  display: block;
}

.contact-form .question-title {
  font-size: 1rem;
  color: var(--primary-color);
}

.contact-form input,
.contact-form select {
  width: 100%;
  padding: 0.5rem;
  border: 2px solid #EEEEEE;
  border-radius: 5px;
  outline: none;
}

.contact-form textarea {
  width: 205%;
  min-height: 5rem;
  max-height: 5rem;
  padding: 0.5rem;
  border: 2px solid #EEEEEE;
  border-radius: 5px;
  outline: none;
}

.error {
  color: red;
  font-size: 1rem;
}

@media (max-width: 1200px) {
  .contact-form textarea {
    width: calc(100% - .5rem);
  }
}

@media (max-width: 800px) {
  .contact-formcontact-form textarea {
    width: calc(100% - .5rem);
  }
}

.contact-submit {
  width: 90%;
  margin: 2rem auto;
  padding: .5rem 2rem;
  display: flex;
  font-size: 1rem;
  border-radius: 1rem 0;
  background-color: var(--primary-color);
  color: var(--text-color);
  text-align: center;
  border: none;
  outline: none;
  transition: background-color .3s ease-in-out;
}

.contact-submit:hover {
  background-color: var(--secondary-color);
}

</style>
