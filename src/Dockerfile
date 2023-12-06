# Use a base image with both C++ and Python
FROM python:3.8

ARG HF_MODEL=ingrenn/whisper-small-zh-TW

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Install C++ build tools and ffmpeg
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    libsndfile1 \
    ffmpeg && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set the working directory to the root of the app
WORKDIR /app

# Copy the entire whisper directory
COPY whisper /app/whisper

# Set the working directory to the whisper directory
WORKDIR /app/whisper

# Build the main example
RUN make

# Install Flask and Gunicorn for the Python app
RUN pip install Flask gunicorn transformers torch numpy

# Clone Openai Whisper (for converting checkpoints to ggml)
RUN git clone https://github.com/openai/whisper /app/openai-whisper && git clone https://huggingface.co/${HF_MODEL} /app/finetuned-model && python ./models/convert-h5-to-ggml.py /app/finetuned-model/ /app/openai-whisper . && rm -fr /app/finetuned-model /app/openai-whisper

# Copy the Flask app to the container (assuming it's in the src directory)
COPY app.py /app/

# Expose the port the app runs on
EXPOSE 5000

# Start the Flask application with Gunicorn, adjust the worker number and timeout as needed
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "--timeout", "300", "--chdir", "/app", "app:app"]
