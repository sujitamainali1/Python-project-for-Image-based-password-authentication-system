import hashlib
import json
import os
from PIL import Image, ImageTk
import tkinter as tk
from tkinter import filedialog

# ANSI color escape codes
ANSI_COLORS = {
    'RED': '\033[91m',
    'BLUE': '\033[94m',
    'GREEN': '\033[92m',
    'YELLOW': '\033[93m',
    'ORANGE': '\033[33m',
    'PURPLE': '\033[95m',
    'RESET': '\033[0m'
}

PASSWORD_FILE = 'passwords.json'

def load_passwords():
    try:
        with open(PASSWORD_FILE, 'r') as file:
            return json.load(file)
    except FileNotFoundError:
        return {}


def enroll_user():
    print("\n" + ANSI_COLORS['YELLOW'] + "=========== Enrollment Phase ============" + ANSI_COLORS['RESET'])
    password = []
    image_paths = []

    root = tk.Tk()
    root.title("Image-Based Password Manager - Enrollment")
    root.geometry("800x400")

    def add_image():
        file_path = filedialog.askopenfilename(filetypes=[("Image files", "*.jpg;*.jpeg")])
        if file_path:
            image_paths.append(file_path)
            img = Image.open(file_path)
            img = img.resize((200, 200), Image.LANCZOS)  # Use LANCZOS filter for high-quality resizing
            photo = ImageTk.PhotoImage(img)
            label = tk.Label(root, image=photo)
            label.image = photo
            label.pack()

    tk.Label(root, text="Select images for your password (minimum 2):").pack()
    tk.Button(root, text="Add Image", command=add_image).pack()
    tk.Button(root, text="Finish Enrollment", command=root.quit).pack()

    root.mainloop()
    root.destroy()

    if len(image_paths) < 2:
        print(ANSI_COLORS['RED'] + "Error: You must select at least 2 images." + ANSI_COLORS['RESET'])
        return enroll_user()

    return image_paths

def authenticate_user(enrolled_password):
    print("\n" + ANSI_COLORS['GREEN'] + "================= Authentication Phase ================" + ANSI_COLORS['RESET'])
    entered_password = []
    image_paths = []

    root = tk.Tk()
    root.title("Image-Based Password Manager - Authentication")
    root.geometry("800x400")

    def add_image():
        file_path = filedialog.askopenfilename(filetypes=[("Image files", "*.jpg;*.jpeg")])
        if file_path:
            image_paths.append(file_path)
            img = Image.open(file_path)
            img = img.resize((200, 200), Image.ANTIALIAS)
            photo = ImageTk.PhotoImage(img)
            label = tk.Label(root, image=photo)
            label.image = photo
            label.pack()

    tk.Label(root, text="Select images for authentication (minimum 2):").pack()
    tk.Button(root, text="Add Image", command=add_image).pack()
    tk.Button(root, text="Finish Authentication", command=root.quit).pack()

    root.mainloop()


    if len(image_paths) < 2:
        print(ANSI_COLORS['RED'] + "Error: You must select at least 2 images." + ANSI_COLORS['RESET'])
        return authenticate_user(enrolled_password)

    return image_paths == enrolled_password


def create_user():
    print("\n" + ANSI_COLORS['PURPLE'] + "=== Create User ===" + ANSI_COLORS['RESET'])
    username = input("\n" + ANSI_COLORS['YELLOW'] +"Enter a username: " + ANSI_COLORS['RESET']).strip()
    if username in passwords:
        print(ANSI_COLORS['RED'] + "Username already exists. Please choose a different username." + ANSI_COLORS['RESET'])
        return

    password = enroll_user()
    hashed_password = hashlib.sha256(''.join(password).encode()).hexdigest()
    passwords[username] = hashed_password
    save_passwords(passwords)
    print(ANSI_COLORS['GREEN'] + "User created successfully." + ANSI_COLORS['RESET'])


def login():
    print("\n" + ANSI_COLORS['PURPLE'] + "===================== Login ====================" + ANSI_COLORS['RESET'])
    username = input("Enter your username: ").strip()
    if username not in passwords:
        print(ANSI_COLORS['RED'] + "Invalid username." + ANSI_COLORS['RESET'])
        return False

    password = enroll_user()
    entered_password = hashlib.sha256(''.join(password).encode()).hexdigest()

    if passwords[username] == entered_password:
        print(ANSI_COLORS['YELLOW'] + "Authentication successful. Welcome, {}!".format(username) + ANSI_COLORS['RESET'])
        return True

    print(ANSI_COLORS['RED'] + "Authentication failed. Please try again." + ANSI_COLORS['RESET'])
    return False

def save_passwords(password_dict):
    with open(PASSWORD_FILE, 'w') as file:
        json.dump(password_dict, file)

#Main Program Starts here.
# Check if passwords file exists, if not create an empty file
if not os.path.exists(PASSWORD_FILE):
    with open(PASSWORD_FILE, 'w') as file:
        file.write('{}')

passwords = load_passwords()

# Project description and disclaimer
print(ANSI_COLORS['YELLOW'] + "Image-Based Password Manager" )
print("This program allows you to create and manage image-based passwords for user authentication.")
print( "!!!!!!!!!!!!!!!!!Please note that this program is for educational purposes only!!!!!!!!!!!!!!" + ANSI_COLORS['RESET'])

while True:

    print("\n" + ANSI_COLORS['BLUE'] + "============= MENU ============" + ANSI_COLORS['RESET'])
    print("\n" + ANSI_COLORS['GREEN'] + "1. Create User" + ANSI_COLORS['RESET'])
    print("\n" + ANSI_COLORS['GREEN'] + "2. Login" + ANSI_COLORS['RESET'])
    print("\n" + ANSI_COLORS['GREEN'] + "3. Exit" + ANSI_COLORS['RESET'])
    choice = input("\n" + ANSI_COLORS['YELLOW'] + "Enter your choice (1-4): " + ANSI_COLORS['RESET'])

    if choice == '1':
        create_user()
    elif choice == '2':
        login()
    elif choice == '3':
        break
    else:
        print(ANSI_COLORS['RED'] + "Invalid choice. Please try again." + ANSI_COLORS['RESET'])

print("\n" + ANSI_COLORS['PURPLE'] + "Thank you for using the program!" + ANSI_COLORS['RESET'])
