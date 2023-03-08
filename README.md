# Frontend

---

## Description

Creation of an online reservation system for a medical clinic.

#### Frontend:

Include a form that requests the following data:

- Name
- DNI (ID)
- Phone
- Email
- Type of appointment: "First consultation" or "review".

It is requested:

- After the user enters the DNI, check via AJAX if it exists in the database, if it exists, the value of the type of
  appointment must be updated to "REVIEW", and if it does not exist, to "FIRST CONSULTATION".
- We must also validate with Javascript that the email entered is correct.

#### Backend:

Requests will be classified in order of arrival and the first available hour will be assigned, taking into account that:

- all days of the week are acceptable for an appointment, except weekends.
- the consultations last one hour and can only be assigned from 10 in the morning to 10 at night.
- As hours are assigned, they must be considered to assign new appointments.
- Two patients cannot be given an appointment at the same time on the same day, nor can “hours” be left empty.

The date and time must be assigned to that appointment, and saved in the database.

It is requested:

1. Implement data model in SQL.
2. Implement reception of data from the form to know if it is the first consultation or not.
3. Implement a system for assigning appointments/hours, considering that there will be concurrence (that is, N requests
   can arrive simultaneously via the Internet).
4. Implement emailing the patient with the appointment with the date, time, and type of appointment.

---

## Vue 3 + Vite

### Development Server

Start the development server on http://localhost:3000. Make sure the port 3000 is free.

```bash
docker compose up -d
```

Stop the development server:

```bash
docker compose up down
```
