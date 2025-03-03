"""
YouTube Video Downloader
Copyright (c) 2025 Dnvr Christ Maniema. Tous droits réservés.

Ce script permet de télécharger des vidéos YouTube en fonction de la qualité choisie par l'utilisateur.
Il prend en charge les systèmes Windows, macOS et Linux.
"""

import os
import customtkinter as ctk
import yt_dlp
from tkinter import messagebox, StringVar, OptionMenu
import platform
import subprocess

# Fonction pour télécharger la vidéo
def download_video():
    url = url_entry.get()
    
    if not url:
        messagebox.showerror("Erreur", "Veuillez entrer une URL valide.")
        return

    try:
        # Chemin où les vidéos seront téléchargées (adapté à tous les systèmes)
        download_folder = os.path.join(os.path.expanduser("~"), "Downloads")
        
        # Options de téléchargement
        ydl_opts = {
            'outtmpl': os.path.join(download_folder, '%(title)s.%(ext)s'),
            'progress_hooks': [progress_hook],
        }

        # Choix du format
        format_choice = format_var.get()
        if format_choice == "Meilleure qualité vidéo":
            ydl_opts['format'] = 'bestvideo+bestaudio/best'
        elif format_choice == "Qualité moyenne vidéo":
            ydl_opts['format'] = 'mp4'
        elif format_choice == "Audio uniquement (MP3)":
            ydl_opts['format'] = 'bestaudio/best'
            ydl_opts['postprocessors'] = [{
                'key': 'FFmpegExtractAudio',
                'preferredcodec': 'mp3',
                'preferredquality': '192',
            }]

        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            ydl.download([url])

        messagebox.showinfo("Succès", f"Téléchargement terminé dans {download_folder}")
        open_download_folder(download_folder)  # Ouvrir le dossier de téléchargement
    except Exception as e:
        messagebox.showerror("Erreur", f"Erreur de téléchargement : {e}")

# Fonction de mise à jour de la barre de progression
def progress_hook(d):
    if d['status'] == 'downloading':
        percent_str = d.get('_percent_str', '0.0%').strip('%')
        try:
            percent = float(percent_str)
            progress_bar.set(percent / 100)
        except ValueError:
            progress_bar.set(0)  # Réinitialise si la conversion échoue

# Fonction pour ouvrir le dossier de téléchargement
def open_download_folder(path):
    """Ouvre le dossier de téléchargement selon l'OS."""
    system = platform.system()
    if system == "Windows":
        os.startfile(path)  # Windows
    elif system == "Darwin":  # macOS
        subprocess.run(["open", path])
    elif system == "Linux":
        subprocess.run(["xdg-open", path])
    else:
        print(f"Système d'exploitation non supporté : {system}")

# Configuration de la fenêtre principale
ctk.set_appearance_mode("dark")  # Mode sombre
ctk.set_default_color_theme("blue")  # Thème bleu

window = ctk.CTk()
window.title("Téléchargeur de Vidéos YouTube")
window.geometry("500x400")

# Titre
title_label = ctk.CTkLabel(window, text="Télécharger une vidéo YouTube", font=("Helvetica", 20))
title_label.pack(pady=20)

# Champ pour l'URL
url_entry = ctk.CTkEntry(window, placeholder_text="Entrez l'URL de la vidéo", width=350)
url_entry.pack(pady=10)

# Menu déroulant pour choisir le format
format_var = StringVar(value="Meilleure qualité vidéo")  # Valeur par défaut
format_options = ["Meilleure qualité vidéo", "Qualité moyenne vidéo", "Audio uniquement (MP3)"]
format_menu = ctk.CTkOptionMenu(window, variable=format_var, values=format_options)
format_menu.pack(pady=10)

# Bouton pour télécharger
download_button = ctk.CTkButton(window, text="Télécharger", command=download_video, fg_color="green", hover_color="darkgreen")
download_button.pack(pady=20)

# Barre de progression
progress_bar = ctk.CTkProgressBar(window, width=300)
progress_bar.pack(pady=10)
progress_bar.set(0)

# Lancer l'interface
window.mainloop()


