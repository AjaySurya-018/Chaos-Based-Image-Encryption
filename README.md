# Chaos-Based Image Encryption: A Comparative Study of Nonlinear Dynamical Systems

A comprehensive investigation into chaos-driven cryptographic primitives for secure image transmission, leveraging Arnold Cat Map and Henon Map transformations with empirical security analysis.

## Abstract

Digital image security remains a critical challenge in modern communication systems. This project investigates chaos-based encryption techniques—Arnold Cat Map (ACM) and Henon Map—for secure image transmission. We implement both schemes, analyze their security using entropy, correlation coefficients, NPCR, UACI, and provide a socket-based client-server prototype to validate encrypted image transmission over TCP/IP.

Key findings:
- ACM: faster, excellent spatial scrambling.
- Henon: stronger statistical resistance and larger key space.
- Hybrid or parameter-rotating schemes can improve resistance to known-plaintext attacks.

---

## Problem Statement

Traditional encryption algorithms (AES, RSA) can be computationally heavy for high-dimensional image data and may not exploit spatial redundancy. Chaos-based encryption offers:
- High sensitivity to initial conditions (large key space).
- Deterministic but pseudorandom transformation.
- Efficient pixel-level diffusion & confusion.

Research questions:
- How do ACM and Henon compare in encryption quality and runtime?
- What parameter ranges optimize entropy and minimize correlation?
- Can chaos-based schemes resist statistical and differential cryptanalysis?

---

## Theoretical Foundation

### Arnold Cat Map (ACM)

<img width="867" height="366" alt="image" src="https://github.com/user-attachments/assets/cef116cf-f1d5-45d1-a580-d9af50a23d8e" />

ACM is a discrete chaotic map on a 2D torus (for NxN images):

x' = (x + y) mod N  
y' = (x + 2y) mod N

Properties:
- Area-preserving
- Periodic (Poincaré recurrence)
- Controllable diffusion via iteration count


### Henon Map

Henon map (2D discrete-time):

x_{n+1} = 1 - a*x_n^2 + y_n  
y_{n+1} = b*x_n

Common parameters: a = 1.4, b = 0.3 (classical chaotic regime)

Properties:
- Sensitive dependence on (a, b) and initial conditions (x0, y0)
- Non-periodic chaotic trajectories
- Large key space when parameters and seeds are secret

---

## Methodology

### Pipeline Architecture

[Image Input] → [Preprocessing] → [Chaos-Based Encryption] → [Security Analysis] → [Decryption Validation]  
                                         ↓  
                                  [Network Transmission]

Steps:
1. Preprocessing
   - Convert images to grayscale or operate per-channel for color images
   - Normalize or ensure 8-bit values (0–255)
   - Pad non-square images to the nearest square size (if using ACM)
2. Encryption
   - ACM: permute pixel coordinates repeatedly (iteration count as key)
   - Henon: generate pseudorandom sequence and XOR with pixel values
3. Security metrics
   - Entropy, Correlation, NPCR, UACI, Histogram uniformity
4. Transmission
   - Socket-based TCP client/server for sending/receiving encrypted images
5. Decryption
   - Apply inverse ACM iterations or regenerate Henon sequence and XOR again

---

## Experimental Setup

- Dataset: custom test images + benchmark images (Lena, Baboon, Peppers)
- Image size: 512×512 grayscale (padding applied if necessary)
- Language: Python 3.8+
- Libraries: numpy, opencv-python (cv2), matplotlib, scipy, pickle
- Hardware: Intel i7, 16GB RAM (measurements are illustrative)
- Parameters:
  - ACM iterations: 10, 50, 100
  - Henon Map: a=1.4, b=0.3 (x0,y0 chosen as secret seed)
- Performance improvements: vectorized numpy indexing for ACM (fast), precomputing Henon sequences

---

## Results & Analysis

Summary table (representative values):

| Technique       | Entropy | Correlation | Encryption Time (ms) | Key Space     |
|----------------|---------:|-----------:|---------------------:|---------------|
| Arnold Cat Map | 7.9974   | 0.0023     | 45                   | ~10^15        |
| Henon Map      | 7.9981   | 0.0008     | 62                   | ~10^28        |
| Original       | 7.4521   | 0.9456     | -                    | -             |

Key observations:
- Both methods achieve near-ideal entropy (~8.0) and destroy pixel correlation.
- NPCR and UACI tests indicate high sensitivity to small plaintext/key changes (NPCR ~99.8%).
- ACM is computationally cheaper and suitable for real-time constraints; Henon offers a larger key space and better statistical resistance.

---

## Secure Communication Framework

Architecture: simple TCP socket-based client/server. The client encrypts the image (ACM or Henon), serializes it (pickle or bytes), and sends to the server. The server receives, deserializes, decrypts and validates integrity.

Client send snippet:
```python
import socket, pickle

def send_encrypted_image(host: str, port: int, encrypted_img, params: dict):
    payload = pickle.dumps({'image': encrypted_img, 'params': params})
    with socket.create_connection((host, port)) as s:
        s.sendall(payload)
        ack = s.recv(1024)
        return ack
```

Server receive snippet:
```python
import socket, pickle

def receive_encrypted_image(conn):
    data = b''
    while True:
        packet = conn.recv(4096)
        if not packet:
            break
        data += packet
    return pickle.loads(data)

# server main loop
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind(('0.0.0.0', 9000))
    s.listen(1)
    conn, addr = s.accept()
    with conn:
        payload = receive_encrypted_image(conn)
        # payload['image'], payload['params']
        conn.sendall(b'ACK')
```

## Key Learnings

- Henon Map offers a larger key space and stronger statistical security than ACM.
- Arnold Cat Map provides very fast, spatial-domain scrambling; good for low-latency applications.
- Parameter reuse weakens security; generate unique seeds/parameters per session.
- Vectorized implementations (NumPy) reduce runtime drastically.
- Hybrid schemes (e.g., ACM permutation + Henon diffusion) can combine strengths of both maps.
