# IIS-chatbot
import json
import re
from datetime import datetime

def load_knowledge_base(file_path):
    try:
        with open(file_path, "r", encoding="utf-8") as file:
            knowledge_base = json.load(file)
        print("Chatbot: Knowledge base loaded successfully.")
        return knowledge_base
    except FileNotFoundError:
        print("Chatbot: Knowledge base file not found. Please provide a valid JSON file.")
        exit()
    except json.JSONDecodeError:
        print("Chatbot: Error decoding the JSON file. Please check the file format.")
        exit()

def normalize_input(user_input):
    return re.sub(r"[^\n]", "", user_input.lower().strip())

def provide_hospital_info(hospital_details):
    print(f"Chatbot: Welcome to {hospital_details['name']}.")
    print(f"Address: {hospital_details['address']}")
    print("Contact Numbers:")
    for key, value in hospital_details['contactNumbers'].items():
        print(f"  - {key.title()}: {value}")
    print("Timings:")
    for key, value in hospital_details['timings'].items():
        print(f"  - {key.title()}: {value}")

def list_departments(departments):
    print("Chatbot: Here are the departments available in our hospital:")
    for dept in departments:
        print(f"  - {dept['name']}: {dept['description']}")

def list_doctors(doctors):
    print("Chatbot: Here is a list of available doctors:")
    print("+--------------------+--------------------+-----------------------------------+")
    print(f"| {'Name':<23} | {'Specialization':<23} | {'Consultation Hours':<23} |")
    print("+--------------------+--------------------+-----------------------------------+")
    for doctor in doctors:
        print(f"| {doctor['name']:<23} | {doctor['specialization']:<23} | {doctor['consultationHours']:<23} |")
    print("+--------------------+--------------------+-----------------------------------+")

def schedule_appointment(doctors, appointments):
    try:
        with open("appointments.json", "r", encoding="utf-8") as file:
            existing_appointments = json.load(file)
    except (FileNotFoundError, json.JSONDecodeError):
        existing_appointments = []

    print("Chatbot: Let's schedule an appointment.")
    print("Chatbot: Please provide your name.")
    patient_name = input("You: ").strip()

    while True:
        print("Chatbot: Which doctor would you like to see?")
        for i, doctor in enumerate(doctors):
            print(f"  {i + 1}. {doctor['name']} ({doctor['specialization']}) - Consultation Hours: {doctor['consultationHours']}")
        try:
            doctor_choice = int(input("You: ").strip()) - 1
            if doctor_choice < 0 or doctor_choice >= len(doctors):
                raise IndexError
            break
        except (ValueError, IndexError):
            print("Chatbot: Invalid choice. Please try again.")

    doctor = doctors[doctor_choice]

    while True:
        print(f"Chatbot: At what time would you like the appointment with {doctor['name']}?")
        appointment_time = input("You: ").strip()

        if any(appt['doctor_name'] == doctor['name'] and appt['time'] == appointment_time and appt['patient_name'].lower() != patient_name.lower() for appt in existing_appointments):
            print(f"Chatbot: {doctor['name']} already has an appointment at {appointment_time} with another patient. Please choose a different time.")
            continue

        consultation_hours = doctor['consultationHours']
        start_time, end_time = consultation_hours.split(" - ")
        
        fmt = "%I:%M %p"
        try:
            requested_time = datetime.strptime(appointment_time, fmt)
            start_time = datetime.strptime(start_time, fmt)
            end_time = datetime.strptime(end_time, fmt)

            if start_time <= requested_time <= end_time:
                break
            else:
                print(f"Chatbot: {doctor['name']} is not available at {appointment_time}. Please choose a time between {consultation_hours}.")
        except ValueError:
            print("Chatbot: Invalid time format. Please use the format 'HH:MM AM/PM'.")

    appointments.append({
        "patient_name": patient_name,
        "doctor_name": doctor["name"],
        "time": appointment_time
    })

    print(f"Chatbot: Your appointment with {doctor['name']} has been scheduled at {appointment_time}.")

def save_appointments(appointments, file_path="appointments.json"):
    try:
        with open(file_path, "r", encoding="utf-8") as file:
            existing_appointments = json.load(file)
    except (FileNotFoundError, json.JSONDecodeError):
        existing_appointments = []

    existing_appointments.extend(appointments)

    with open(file_path, "w", encoding="utf-8") as file:
        json.dump(existing_appointments, file, indent=4)
    print("Chatbot: Appointments have been saved.")

def chatbot(knowledge_base):
    appointments = []
    print("Chatbot: Hi! Welcome to the Hospital Reception Chatbot. How can I assist you today?")
    while True:
        inp = input("You: ").strip().lower()
        if "hospital" in inp or "info" in inp:
            print("\nChatbot: Here is the information about our hospital:\n")
            provide_hospital_info(knowledge_base["hospitalDetails"])
        elif any(word in inp for word in ["departments", "dept", "dept.", "Dept.", "Dept", "department", "Department", "depart", "Depart", "depart."]):
            print("\nChatbot: Here is the list of departments available in our hospital:\n")
            list_departments(knowledge_base["departments"])
        elif any(word in inp for word in ["doctors", "doctor", "doc", "Doctor", "Doc", "doctor."]):
            print("\nChatbot: Here is the list of doctors available in our hospital:\n")
            list_doctors(knowledge_base["doctors"])
        elif any(word in inp for word in ["schedule", "appointment", "appoint"]):
            schedule_appointment(knowledge_base["doctors"], appointments)
        elif any(word in inp for word in ["exit", "quit", "Exit", "Quit", "EXIT", "QUIT"]):
            save_appointments(appointments)
            print("Chatbot: Thank you for using the chatbot. Have a great day!")
            break
        else:
            print("Chatbot: Invalid Input. Please try again.")

if __name__ == "__main__":
    knowledge_base_file = "knowledge_base.json"
    knowledge_base = load_knowledge_base(knowledge_base_file)
    chatbot(knowledge_base)
