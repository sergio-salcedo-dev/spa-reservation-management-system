<template>
  <h1 class="p-5">Reservations</h1>
  <h2 class="p-2">Book an appointment</h2>
  <main class="p-5 main">
    <section class="mx-auto">
      <form class="col-md-12 my-2" @submit.prevent="checkDni">
        <label for="dni" class="form-label">DNI</label>
        <input
          type="text"
          class="form-control"
          id="dni"
          v-model.trim="formData.dni"
          placeholder="04123456A"
          required
          @blur.prevent="checkDni"
          @focus="message = ''"
        />
        <small v-if="errors.dni" class="d-block mt-1 text-danger text-small">{{ errors.dni }}</small>
      </form>

      <form v-if="isDniValidated" class="row g-3 mt-2" @submit.prevent="submitForm">
        <div class="col-md-12 my-2">
          <label for="full-name" class="form-label">Full name</label>
          <input
            type="text"
            class="form-control"
            :readonly="patientId"
            id="full-name"
            v-model.trim="formData.fullName"
            placeholder="John Doe"
            required
          />
          <small v-if="errors.fullName" class="mt-1 text-danger text-small">{{ errors.fullName }}</small>
        </div>

        <div class="col-md-12 my-2">
          <label for="email" class="form-label">Email</label>
          <input
            type="email"
            class="form-control"
            id="full-name"
            v-model.trim="formData.email"
            :readonly="patientId"
            placeholder="John Doe"
            required
          />
          <small v-if="errors.email" class="mt-1 text-danger text-small">{{ errors.email }}</small>
        </div>

        <div class="col-md-12 my-2">
          <label for="phone" class="form-label">Phone</label>
          <input
            type="tel"
            class="form-control"
            id="phone"
            aria-describedby="phoneFeedback"
            placeholder="+34600123456"
            v-model.trim="formData.phone"
            :readonly="patientId"
            required
          />
          <small v-if="errors.phone" class="mt-1 text-danger text-small">{{ errors.phone }}</small>
        </div>

        <div class="col-md-12 my-2">
          <label for="appointmentType" class="form-label">Appointment type</label>
          <select
            class="form-select"
            id="appointmentType"
            aria-describedby="appointmentTypeFeedback"
            v-model="formData.appointmentType"
            required
          >
            <option selected disabled value="">Choose...</option>
            <option>{{ formData.appointmentType }}</option>
          </select>
          <small v-if="errors.appointmentType" class="mt-1 text-danger text-small">{{ errors.appointmentType }}</small>
        </div>

        <div class="col-12 mt-5">
          <button class="btn btn-primary" type="submit">Book your appointment</button>
        </div>
      </form>

      <article
        v-if="message"
        class="my-3 alert"
        :class="[isAppointmentBooked ? 'alert-success' : 'alert-danger']"
        role="alert"
        v-html="message"
      />
    </section>
  </main>
</template>

<script setup>
import { ref } from 'vue';

const formData = ref({
  dni: '',
  fullName: '',
  phone: '',
  email: '',
  appointmentType: '',
});

const API_ENDPOINT = 'http://localhost/api';
const patientId = ref(0);
const isDniValidated = ref(false);
const canValidateForm = ref(false);
const isAppointmentBooked = ref(false);
const message = ref('');
const errors = ref({});

const resetDni = () => {
  formData.value.dni = '';
  isDniValidated.value = false;
};

const reset = () => {
  errors.value = {};
  isAppointmentBooked.value = false;
  canValidateForm.value = false;
  message.value = '';
  isDniValidated.value = false;
  patientId.value = 0;
  formData.value.fullName = '';
  formData.value.email = '';
  formData.value.phone = '';
  formData.value.appointmentType = '';
};

const checkDni = async () => {
  reset();
  const dni = formData.value.dni;

  // Validate DNI
  if (!dni) {
    errors.value.dni = 'DNI is required';
  } else if (!/^[0-9]{8}[A-Za-z]$/.test(dni)) {
    errors.value.dni = 'Invalid DNI';
  } else {
    isDniValidated.value = true;

    try {
      // make API call to check if DNI exists in the backend
      const response = await fetch(`${API_ENDPOINT}/patients/search/${dni}`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Accept: 'application/json',
        },
      });
      const { data: patient } = await response.json();

      if (response.status === 200) {
        canValidateForm.value = true;
        patientId.value = patient?.id;
        formData.value.dni = patient?.dni;
        formData.value.fullName = patient?.full_name;
        formData.value.phone = patient?.phone;
        formData.value.email = patient?.email;
        formData.value.appointmentType = 'Revision';
      } else if (response.status === 404) {
        formData.value.appointmentType = 'First appointment';
      } else {
        formData.value.appointmentType = '';
        throw new Error();
      }
    } catch (error) {
      console.log(error);
      message.value = `Error while checking your DNI`;
    }
  }
};

const createPatient = async () => {
  if (patientId.value) {
    return;
  }

  try {
    const request = {
      dni: formData.value.dni,
      full_name: formData.value.fullName,
      phone: formData.value.phone,
      email: formData.value.email,
    };
    const response = await fetch(`${API_ENDPOINT}/patients/`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
      },
      body: JSON.stringify(request),
    });

    const data = await response.json();
    if (response.status === 422) {
      message.value = `Validation Error. ${data?.message}`;
    } else if (response.status === 201) {
      canValidateForm.value = true;
      patientId.value = data.data.id;
    } else {
      message.value = `Server Error. ${data?.message}`;
    }
  } catch (error) {
    console.log(error);
    message.value = 'Something went wrong creating the new patient';
  }
};

const createAppointment = async () => {
  // Send form data to backend
  const request = {
    appointment_type: formData.value.appointmentType.toLocaleLowerCase(),
    dni: formData.value.dni,
  };

  try {
    const response = await fetch(`${API_ENDPOINT}/appointments`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
      },
      body: JSON.stringify(request),
    });

    const data = await response.json();

    if (response.status === 201) {
      isAppointmentBooked.value = true;
      const appointmentData = data.data;
      const dayOfWeek = appointmentData.dayOfWeek;
      const date = appointmentData.date;
      const time = appointmentData.time;
      const duration = appointmentData.duration;
      const type = appointmentData.appointmentType;
      const appointmentDetails = `Appointment details: ${date}, ${dayOfWeek} at ${time}h, duration: ${duration} h., type: ${type}`;

      message.value = `Appointment booked successfully. <br><br> ${appointmentDetails}. <br><br>Appointment details has been sent to your email.`;
    } else if (response.status === 422) {
      message.value = `Validation Error. ${data?.message}`;
    } else {
      message.value = `Server Error. ${data?.message}`;
    }
  } catch (error) {
    console.log(error);
    message.value = 'Error while booking your appointment';
  }
};

const submitForm = async () => {
  await createPatient();

  if (!canValidateForm.value) {
    return;
  }

  isAppointmentBooked.value = false;
  const isFormValid = validateForm();

  if (!isFormValid) {
    message.value = `The form has errors`;

    return;
  }

  await createAppointment();
  resetDni();
};

const validateForm = () => {
  // Reset errors
  errors.value = {};

  // Validate name
  if (!formData.value.fullName) {
    errors.value.fullName = 'Full name is required';
  }

  // Validate email
  if (!formData.value.email) {
    errors.value.email = 'Email is required';
  } else if (!/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/.test(formData.value.email)) {
    errors.value.email = 'Invalid email';
  }

  // Validate phone
  if (!formData.value.phone) {
    errors.value.phone = 'Phone is required';
  }

  // Validate appointmentType
  if (!formData.value.appointmentType) {
    errors.value.appointmentType = 'Appointment type is required';
  }

  return !Object.keys(errors.value).length;
};
</script>

<style scoped>
.main {
  min-width: 360px;
  max-width: 1400px;
  width: 50%;
}
</style>
