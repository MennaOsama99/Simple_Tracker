# Simple_Tracker
#tacking system simple and cute using C++ code

# C++ KCF Tracker (RISC-V RVV Accelerated)

This package includes a C++ implementation of several tracking methods based on the Kernelized Correlation Filter (KCF) [1, 2]. 

This repository has been specifically modified and extended for the **RISC-V SI-Vision Academy Project**. The primary objective of this fork is to accelerate key computational kernels of the KCF tracking pipeline using **RISC-V Vector (RVV)** intrinsics, while maintaining a scalar fallback for host (x86/ARM) compatibility.

[1] J. F. Henriques, R. Caseiro, P. Martins, J. Batista, "High-Speed Tracking with Kernelized Correlation Filters", TPAMI 2015.  
[2] J. F. Henriques, R. Caseiro, P. Martins, J. Batista, "Exploiting the Circulant Structure of Tracking-by-detection with Kernels", ECCV 2012.

**Original Authors:** Joao Faro, Christian Bailer, Joao F. Henriques  
**Original Contacts:** joaopfaro@gmail.com, Christian.Bailer@dfki.de, henriques@isr.uc.pt  
**Original Institution:** Institute of Systems and Robotics - University of Coimbra / Department of Augmented Vision DFKI  

**Project Contributors:** Menna O. Sayed / student3  
**Project Context:** RISC-V Academy Project (Milestones 1-4)

---

### 🚀 Key Modifications in This Repository

1. **OpenCV 4.x Compatibility:** Updated deprecated OpenCV 2/3 constants (e.g., `CV_BGR2GRAY` → `cv::COLOR_BGR2GRAY`) and fixed legacy `IplImage` casting issues to ensure seamless compilation on modern Ubuntu systems.
2. **RVV Kernel Acceleration (Student 3 Scope):** 
   - **K5 (Gaussian Correlation & Complex Ops):** Optimized frequency-domain element-wise complex multiplication using RVV Stride Loads (`vlse32`/`vsse32`) and Fused Multiply-Add (`vfmacc`/`vfnmsac`) instructions, eliminating the need for slow scalar array splitting.
   - **K6 (Response-Map Argmax & Subpixel Peak):** Replaced `cv::minMaxLoc` with a hybrid RVV vector reduction (`vfredmax`) and scalar subpixel quadratic interpolation for highly efficient peak detection.
3. **Conditional Compilation:** The code uses `#ifdef __riscv_vector` to automatically switch between the optimized RVV kernels (when cross-compiled for RISC-V) and the original OpenCV scalar fallback (when compiled on a host PC).
4. **Headless Execution Support:** Added `test_headless.cpp` to allow tracking validation and video output generation (`output.avi`) in environments without a graphical display server (e.g., WSL, remote servers, or gem5).

---

### 📂 Algorithms Included

- **KCF (HOG):** Kernelized Correlation Filter on HOG features (Default).
- **KCF (Lab):** KCF on HOG and quantized CIE-Lab color features.
- **CSK (Bonus):** Circulant Structure with Kernels using raw grayscale features (single-channel).

---

### ⚙️ Compilation Instructions

#### Option A: Host PC Compilation (Scalar Fallback)
*Requires OpenCV 4.x and standard `g++`.*
```bash
mkdir build && cd build
cmake ..
make


# Compile the headless test with RVV extensions enabled
riscv64-linux-gnu-g++ -O3 -march=rv64gcv -mabi=lp64d -D__riscv_vector \
    test_headless.cpp \
    ../src/kcftracker.cpp \
    ../src/fhog.cpp \
    -I../src \
    $(pkg-config --cflags --libs opencv4) \
    -lpthread -lm -o test_headless_rvv
    
    
    #The executable KCF can be called with the following options:
    ./KCF [OPTION_1] [OPTION_2] [...]
    
    
    #Generates an output.avi file with the tracked bounding boxes drawn, bypassing the need for cv::imshow.
    cd build
# Compile the headless test (if not already done)
g++ -O3 -o test_headless test_headless.cpp ../src/kcftracker.cpp ../src/fhog.cpp -I../src $(pkg-config --cflags --libs opencv4) -lpthread -lm

# Run on a sample video
./test_headless test.mp4


#To measure true hardware speedup (cycles/instructions), run the standalone kernel benchmarks in gem5 Syscall Emulation (SE) mode:
# From the gem5 root directory
./build/RISCV/gem5.opt configs/deprecated/example/se.py \
    --cpu-type=O3CPU \
    --caches \
    -c /path/to/KCFcpp/build/benchmark_kernels
    
    
    

