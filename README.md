# Hospital-Management-System-
Hospital Management System designed to digitally manage hospital information, patient and doctor records, and administrative activities. This academic project demonstrates software development, data management, and system design concepts while providing a foundation for efficient and organized hospital operations.
from flask import Flask, render_template, request, redirect, url_for
from pymongo import MongoClient
from bson.objectid import ObjectId
from dotenv import load_dotenv
import os

load_dotenv()

app = Flask(__name__)

# MongoDB Connection
client = MongoClient(os.getenv("MONGO_URI"))
db = client["hospital_management"]

patients = db["patients"]
doctors = db["doctors"]
appointments = db["appointments"]


# ---------------- HOME ----------------

@app.route("/")
def index():
    patient_count = patients.count_documents({})
    doctor_count = doctors.count_documents({})
    appointment_count = appointments.count_documents({})

    return render_template(
        "index.html",
        patient_count=patient_count,
        doctor_count=doctor_count,
        appointment_count=appointment_count
    )


# ---------------- PATIENTS ----------------

@app.route("/patients")
def patient_list():
    all_patients = patients.find().sort("_id", -1)
    return render_template("patients.html", patients=all_patients)


@app.route("/patients/add", methods=["GET", "POST"])
def add_patient():

    if request.method == "POST":

        patient = {
            "name": request.form["name"],
            "age": int(request.form["age"]),
            "gender": request.form["gender"],
            "phone": request.form["phone"],
            "email": request.form["email"],
            "address": request.form["address"],
            "disease": request.form["disease"]
        }

        patients.insert_one(patient)

        return redirect(url_for("patient_list"))

    return render_template("add_patient.html")


# ---------------- DELETE PATIENT ----------------

@app.route("/patients/delete/<id>")
def delete_patient(id):

    patients.delete_one({
        "_id": ObjectId(id)
    })

    return redirect(url_for("patient_list"))


# ---------------- DOCTORS ----------------

@app.route("/doctors")
def doctor_list():

    all_doctors = doctors.find().sort("_id", -1)

    return render_template(
        "doctors.html",
        doctors=all_doctors
    )


@app.route("/doctors/add", methods=["POST"])
def add_doctor():

    doctor = {
        "name": request.form["name"],
        "specialization": request.form["specialization"],
        "phone": request.form["phone"],
        "email": request.form["email"]
    }

    doctors.insert_one(doctor)

    return redirect(url_for("doctor_list"))


# ---------------- APPOINTMENTS ----------------

@app.route("/appointments")
def appointment_list():

    all_appointments = appointments.find().sort("_id", -1)

    return render_template(
        "appointments.html",
        appointments=all_appointments
    )


@app.route("/appointments/add", methods=["POST"])
def add_appointment():

    appointment = {
        "patient_name": request.form["patient_name"],
        "doctor_name": request.form["doctor_name"],
        "date": request.form["date"],
        "time": request.form["time"],
        "reason": request.form["reason"],
        "status": "Scheduled"
    }

    appointments.insert_one(appointment)

    return redirect(url_for("appointment_list"))


# ---------------- RUN APPLICATION ----------------

if __name__ == "__main__":
    app.run(debug=True)
