# Use a lean, official Python image as the base
FROM python:3.10-slim

# Set the working directory inside the container
WORKDIR /app

# Install essential system packages. ffmpeg is required by yt-dlp for merging video/audio.
# Using --no-install-recommends keeps the image size small.
RUN apt-get update && apt-get install -y --no-install-recommends ffmpeg && rm -rf /var/lib/apt/lists/*

# Copy the requirements file into the container
COPY requirements.txt .
# Install the Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Install/Update to the latest version of yt-dlp
RUN pip install --no-cache-dir -U yt-dlp

# Copy the rest of our application code into the container
COPY . .

# Create the downloads directory and set permissions
# 'chown' changes ownership to a less-privileged user.
RUN mkdir -p downloads && chown -R www-data:www-data /app

# SECURITY: Switch from the 'root' user to a standard, non-privileged 'www-data' user.
# This is a critical security best practice.
USER www-data

# Expose the port that Gunicorn will run on inside the container
EXPOSE 8000

# The command to run when the container starts.
# It uses Gunicorn, a professional-grade Python web server.
# --bind 0.0.0.0:8000: Listen on all network interfaces inside the container.
# --workers 2: Run 2 parallel processes of the app for better performance.
# app:app: Tells Gunicorn to run the 'app' object from the 'app.py' file.
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "2", "app:app"]