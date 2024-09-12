# Obfuscating_python_code_with_pyarmor

    # Stage 1: Build the application (obfuscation)
    FROM python:3.11-slim-buster AS obfuscator
    
    # Set environment variables
    ENV PYTHONDONTWRITEBYTECODE 1
    ENV PYTHONUNBUFFERED 1
    
    # Set the working directory
    WORKDIR /app
    
    # Install git and necessary tools for obfuscation
    RUN apt-get update && apt-get install -y git
    
    # Copy all necessary files to the container
    COPY . ./
    
    # Install PyArmor and dependencies from requirements.txt
    RUN pip install pyarmor
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Obfuscate the entire application recursively, including all subdirectories
    RUN pyarmor gen -r -O /app/dist main.py
    
    # Debugging: Ensure all files are included after obfuscation
    RUN ls -la /app/dist
    
    # Stage 2: Create the final runtime image
    FROM python:3.11-slim-buster AS runner
    
    # Install git (needed for git-based dependencies in requirements.txt)
    RUN apt-get update && apt-get install -y git
    
    
    # Set environment variables
    ENV PYTHONDONTWRITEBYTECODE 1
    ENV PYTHONUNBUFFERED 1
    
    # Set the working directory
    WORKDIR /app
    
    # Copy the obfuscated files from the build stage
    COPY --from=obfuscator /app/dist/ ./
    
    # remember in the 2nd stage you require all files that would use obfuscated main.py. that is why you first get the obfuscated main.py from stage 1 to stage 2 and all other remaining files from stage 1 to stag 2.
    # Explicitly copy models.py or models directory to the final directory
    COPY --from=obfuscator /app/models.py ./
    # or if you have a models directory
    # COPY --from=obfuscator /app/models ./models
    
    COPY --from=obfuscator /app/helper.py ./
    COPY --from=obfuscator /app/environment.py ./
    # Copy the requirements.txt file
    COPY --from=obfuscator /app/requirements.txt ./
    COPY --from=obfuscator /app/prompt.txt ./
    
    # Install dependencies inside the final image
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Debug step: Check the contents of the /app/ folder
    RUN ls -la /app
    
    # Expose the application port
    EXPOSE 8000
    
    # Command to run the FastAPI application
    CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

## Obfuscating with pyarmon 7.7.4 verison

        # Stage 1: Build the application (obfuscation and packaging)
        FROM python:3.10-slim-buster AS obfuscator
        
        # Set environment variables
        ENV PYTHONDONTWRITEBYTECODE 1
        ENV PYTHONUNBUFFERED 1
        
        # Set the working directory
        WORKDIR /app
        
        # Install git, binutils, and necessary tools for obfuscation and packaging
        RUN apt-get update && apt-get install -y git
        
        # Copy all necessary files to the container
        COPY . ./
        
        # Install PyArmor and PyInstaller, and dependencies from requirements.txt
        RUN pip install pyarmor==7.7.4
        RUN pip install --no-cache-dir -r requirements.txt
        
        # Obfuscate the entire application recursively, including all subdirectories
        RUN pyarmor obfuscate -O obf main.py
        RUN ls -la /app/obf
        
        # Stage 2: Create the final runtime image
        FROM python:3.10-slim-buster AS runner
        
        # Install git (needed for git-based dependencies in requirements.txt)
        RUN apt-get update && apt-get install -y git
        
        # Set environment variables
        ENV PYTHONDONTWRITEBYTECODE 1
        ENV PYTHONUNBUFFERED 1
        
        # Set the working directory
        WORKDIR /app
        
        # Copy the obfuscated files from the build stage
        COPY --from=obfuscator /app/obf/ ./
        RUN ls -la /app
        COPY --from=obfuscator /app/requirements.txt ./
        
        # Install dependencies inside the final image
        RUN pip install --no-cache-dir -r requirements.txt
        
        # Debug step: Check the contents of the /app/ folder
        RUN ls -la /app
        
        # Expose the application port
        EXPOSE 8000
        
        # Command to run the FastAPI application
        CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

## with pyObfuscator module

    # Stage 1: Build the application (obfuscation and packaging)
    FROM python:3.11-slim-buster AS obfuscator
        
    # Set environment variables
    ENV PYTHONDONTWRITEBYTECODE 1
    ENV PYTHONUNBUFFERED 1
        
    # Set the working directory
    WORKDIR /app
        
    # Install git, binutils, and necessary tools for obfuscation and packaging
    RUN apt-get update && apt-get install -y git
        
    # Copy all necessary files to the container
    COPY . ./
    
    # Step 5: Install PyObfuscator
    RUN pip install PyObfuscator
    RUN pip install --no-cache-dir -r requirements.txt
    # Step 6: Obfuscate the Python script
    RUN PyObfuscator main.py -o main_obf.py
    RUN ls -la /app
    
    # Stage 2: Create the final runtime image
    FROM python:3.11-slim-buster AS runner
        
    # Install git (needed for git-based dependencies in requirements.txt)
    RUN apt-get update && apt-get install -y git
        
    # Set environment variables
    ENV PYTHONDONTWRITEBYTECODE 1
    ENV PYTHONUNBUFFERED 1
        
    # Set the working directory
    WORKDIR /app
        
    # Copy the obfuscated files from the build stage
    COPY --from=obfuscator /app/ ./
    #RUN ls -la /app
    COPY --from=obfuscator /app/requirements.txt ./
        
    # Install dependencies inside the final image
    RUN pip install --no-cache-dir -r requirements.txt
        
    # Debug step: Check the contents of the /app/ folder
    #RUN ls -la /app
        
    # Expose the application port
    EXPOSE 8000
        
    # Command to run the FastAPI application
    CMD ["uvicorn", "main_obf:app", "--host", "0.0.0.0", "--port", "8000"]


### With pyminifier obfuscate

        # Use python:3.11-slim as base image
        FROM python:3.11-slim-buster
        
        # Set environment variables
        ENV PYTHONDONTWRITEBYTECODE 1
        ENV PYTHONUNBUFFERED 1
        
        # Set the working directory inside the container
        WORKDIR /app
        
        # Copy the current directory (with the Python script) to the container
        COPY . .
        
        # Install git and necessary packages
        RUN apt-get update && apt-get install -y git
        
        # Install dependencies
        RUN pip install --no-cache-dir -r requirements.txt
        RUN pip3 install --upgrade pip setuptools==57.5.0
        # Install Pyminifier
        RUN pip install pyminifier
        
        # Create the input directory and move main.py into it
        #RUN mkdir -p input
        #RUN cp main.py input/main.py
        
        # Obfuscate the script using Pyminifier
        RUN pyminifier --obfuscate --replacement-length=4 -o main_obf.py main.py
        # Remove the input directory
        #RUN rm -rvf input
        
        # Verify the obfuscated file exists
        #RUN ls -l /app/obfuscated/
        
        # Move the obfuscated script to the root directory
        #RUN mv /app/obfuscated/main_obf.py /app/
        
        EXPOSE 8000
        
        # Run the obfuscated Python script
        CMD ["uvicorn", "main_obf:app", "--host", "0.0.0.0", "--port", "8000"]

# with Intensio Obfuscation tool

        # Use python:3.11-slim as base image
        FROM python:3.11-slim-buster
        
        # Set environment variables
        ENV PYTHONDONTWRITEBYTECODE 1
        ENV PYTHONUNBUFFERED 1
        
        # Set the working directory inside the container
        WORKDIR /app
        
        # Copy the current directory (with the Python script) to the container
        COPY . .
        
        # Install git and necessary packages
        RUN apt-get update && apt-get install -y git
        
        # Install dependencies
        RUN pip install --no-cache-dir -r requirements.txt
        
        # Create the input directory and move main.py into it
        RUN mkdir -p input
        RUN cp main.py input/main.py
        
        # Clone the Intensio-Obfuscator repository
        # RUN git clone https://github.com/Hnfull/Intensio-Obfuscator.git
        
        # Install the obfuscator dependencies
        # RUN pip install -r Intensio-Obfuscator/requirements.txt
        
        # Install Intensio-Obfuscator as a Python package
        RUN pip install intensio-obfuscator
        
        # Obfuscate the script with the required argument
        RUN intensio_obfuscator -i input -o obfuscated -mlen medium -rts -ps -rth -ind 4
        
        # Remove the input directory
        RUN rm -rvf input
        
        # Verify the obfuscated file exists
        RUN ls -l /app/obfuscated/
        RUN cp /app/obfuscated/main.py /app/obfuscated/main_obf.py
        # Move the obfuscated script to the root directory
        RUN mv /app/obfuscated/main_obf.py /app/
        
        EXPOSE 8000
        
        # Run the obfuscated Python script
        CMD ["uvicorn", "main_obf:app", "--host", "0.0.0.0", "--port", "8000"]


