# Dockerfile for our Flask application

# We are about to turn a folder of Python files into a single, runnable artifact
# that behaves the same on our laptop, on a cloud VM, or inside a Kubernetes
# cluster. The instructions below build that artifact layer by layer, starting
# from a bare-bones Linux image and ending with our own code spinning under
# Gunicorn. 

# ============================================================================
# 📦 BASE IMAGE
# ============================================================================

# Every container begins as a copy of some other image. We want one that
# already has Python 3.11 installed, but we do not want the extra weight of
# compilers, man pages, or other tools we will never touch. The official
# python:3.11-slim image gives us exactly that, a stripped-down Debian base
# with Python ready to go. 
FROM python:3.11-slim

# ============================================================================
# 🏷️ METADATA
# ============================================================================

# Labels do not change how the image runs, yet they are handy when we have
# dozens of images floating around a registry and we need to remember who
# built what and why.
LABEL maintainer="danieljbf02@gmail.com"
LABEL description="Self-healing Flask app for K8s demo"

# ============================================================================
# 📂 WORKING DIRECTORY
# ============================================================================

# Here, we create and move into the /app directory. From this point on, every 
# command  will assume that we are working inside this folder, just as if we 
# had run `cd /app` in the terminal.
WORKDIR /app

# ============================================================================
# ⚙️ INSTALLING DEPENDENCIES
# ============================================================================

# When we build a Docker image, we are really stacking a series of layers on top 
# of each other. Every instruction in the Dockerfile that can change the 
# filesystem (`COPY`, `RUN`, `ADD`, …) creates a new layer, and once a layer is 
# built, it becomes immutable. This means that if even a single byte changes in 
# one layer, Docker has to rebuild that layer and every layer that comes after 
# it from scratch. This is why the order of instructions in a Dockerfile has 
# such a big impact on how fast our builds run. We want to structure the file so 
# that layers that rarely change come first, and layers that change often come 
# later. This helps Docker reuse as many cached layers as possible between builds.
# For a Python project, our dependencies usually change far less often than our 
# application code. The set of third-party packages listed in the 
# `requirements.txt` file might stay the same for weeks, while the source code 
# changes every day, so it makes sense to install the dependencies in their own 
# layer before we copy the rest of the project. As long as the `requirements.txt` 
# file does not change, Docker will simply reuse the pre-built layer that already 
# contains the installed packages, turning a multi-minute `pip install` into a 
# sub-second cache hit.
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

# ============================================================================
# 💻 APPLICATION CODE
# ============================================================================

COPY main.py .

# ============================================================================
# 🔒 SECURITY
# ============================================================================

RUN useradd -m -s /bin/bash appuser && \
    chown -R appuser:appuser /app

USER appuser

# ============================================================================
# 🌐 RUNTIME CONFIGURATION
# ============================================================================

EXPOSE 5000

ENV APP_VERSION=1.0.0
ENV APP_ENV=production
ENV PORT=5000

# ============================================================================
# ❤️ HEALTH CHECK
# ============================================================================

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"

# ============================================================================
# 🚀 THE STARTUP COMMAND
# ============================================================================

CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:5000", "--access-logfile", "-", "--error-logfile", "-", "main:app"]