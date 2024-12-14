# Use a slim Python base image for reduced size and compatibility
FROM python:3.8-slim

# Set the working directory in the container
WORKDIR /app

# Install system dependencies required for PyTorch and other libraries
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    gfortran \
    libopenblas-dev \
    liblapack-dev \
    libstdc++6 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy only the requirements file to leverage Docker's layer caching
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt \
    -f https://download.pytorch.org/whl/cpu/torch_stable.html

# Copy the rest of the application files
COPY . .

# Expose the port Flask runs on
EXPOSE 8001

# Set the environment variable for Flask
ENV FLASK_APP=server.py
ENV FLASK_RUN_HOST=0.0.0.0

# Run the Flask application
CMD ["flask", "run"]
