### create a new File `build_all_runtimes.sh`
```bash
#!/bin/bash
set -e

# --- CONFIGURATION ---
# This is the master plan for our environments.
# Each language gets its own directory and its own specific set of packages.
declare -A LANGUAGES
LANGUAGES=(
    [c-v11]="bullseye:build-essential"
    [cpp-v11]="bullseye:build-essential g++"
    [python-v3.9]="bullseye:python3 python3-pip"
    [go-v1.21]="bullseye:golang-go"
    [javascript-v18]="bullseye:nodejs"
)

# The base directory where all our separate, clean environments will live.
RUNTIME_BASE_DIR="/opt/evalx/runtimes"

# --- SCRIPT LOGIC ---

echo "--- Starting Build Process for All Language Runtimes ---"
sudo mkdir -p "$RUNTIME_BASE_DIR"

# Loop through each language defined in our configuration.
for lang_key in "${!LANGUAGES[@]}"; do
    config=${LANGUAGES[$lang_key]}
    DEBIAN_RELEASE=$(echo "$config" | cut -d':' -f1)
    PACKAGES=$(echo "$config" | cut -d':' -f2)
    
    # Each language gets its own dedicated, isolated directory. THIS IS THE FIX.
    CHROOT_PATH="${RUNTIME_BASE_DIR}/${lang_key}"

    echo ""
    echo "--- Building Environment for: ${lang_key} in ${CHROOT_PATH} ---"
    echo "--- Debian Release: ${DEBIAN_RELEASE}, Packages: ${PACKAGES} ---"

    # 1. Clean up any old environment and create a fresh, empty directory.
    if [ -d "$CHROOT_PATH" ]; then
        echo "Removing existing directory..."
        sudo rm -rf "$CHROOT_PATH"
    fi
    sudo mkdir -p "$CHROOT_PATH"

    # 2. Use debootstrap to create a minimal but complete Debian filesystem.
    # This is like laying the foundation for a new, clean workshop.
    echo "Creating minimal Debian filesystem with debootstrap..."
    sudo debootstrap --variant=minbase "$DEBIAN_RELEASE" "$CHROOT_PATH" http://deb.debian.org/debian/

    # 3. Chroot into the new environment to install ONLY the required tools.
    # We enter the workshop and place only the specific tools needed for this one job.
    echo "Installing language-specific packages: ${PACKAGES}..."
    sudo chroot "$CHROOT_PATH" /bin/bash <<EOF
set -e
export DEBIAN_FRONTEND=noninteractive

# Update package lists inside the new environment
apt-get update -qq

# Install ONLY the packages for this specific language
apt-get install -y -qq --no-install-recommends ${PACKAGES}

# Clean up to keep the final environment small
apt-get clean
rm -rf /var/lib/apt/lists/*
EOF

    echo "✅ Environment for ${lang_key} created successfully."
done

echo ""
echo "--- All Language Runtimes Built Successfully ---"
echo "Next Step: Update the 'chroot_path' in your.toml configuration files."
```

#### Give the permission and run it 
```bash
chmod +x build_all_runtimes.sh
sudo ./build_all_runtimes.sh
```


```bash
sudo bash -c '
# Add essential libraries to ALL runtimes
for runtime in /opt/evalx/runtimes/*; do
    if [ -d "$runtime" ]; then
        echo "Fixing: $runtime"
        
        # Copy essential math and system libraries
        sudo cp -r /lib/x86_64-linux-gnu/libm* $runtime/lib/x86_64-linux-gnu/ 2>/dev/null || true
        sudo cp -r /usr/lib/x86_64-linux-gnu/* $runtime/usr/lib/x86_64-linux-gnu/ 2>/dev/null || true
        sudo cp -r /usr/include/* $runtime/usr/include/ 2>/dev/null || true
        
        # Ensure libstdc++ is available for C++
        sudo cp /usr/lib/x86_64-linux-gnu/libstdc++* $runtime/usr/lib/x86_64-linux-gnu/ 2>/dev/null || true
    fi
done
echo "✅ All runtimes updated with essential libraries"
'
[sudo] password for gunasekhar: 
Fixing: /opt/evalx/runtimes/cpp-v11
Fixing: /opt/evalx/runtimes/c-v11
Fixing: /opt/evalx/runtimes/go-v1.21
Fixing: /opt/evalx/runtimes/javascript-v18
Fixing: /opt/evalx/runtimes/python-v3.9
✅ All runtimes updated with essential libraries
```

--- 
## For verification 
```bash
# Check C runtime
echo "=== Checking C Runtime ==="
sudo ls -la /opt/evalx/runtimes/c-v11/lib/x86_64-linux-gnu/libm* 2>/dev/null | head -5
sudo ls -la /opt/evalx/runtimes/c-v11/usr/lib/x86_64-linux-gnu/libstdc++* 2>/dev/null | head -3

# Check C++ runtime  
echo "=== Checking C++ Runtime ==="
sudo ls -la /opt/evalx/runtimes/cpp-v11/lib/x86_64-linux-gnu/libm* 2>/dev/null | head -5
sudo ls -la /opt/evalx/runtimes/cpp-v11/usr/lib/x86_64-linux-gnu/libstdc++* 2>/dev/null | head -3

# Check Python runtime
echo "=== Checking Python Runtime ==="
sudo ls -la /opt/evalx/runtimes/python-v3.9/usr/lib/x86_64-linux-gnu/ 2>/dev/null | head -5
```


```bash
# Test if C runtime can load math library
echo "=== Testing Math Library Loading ==="
sudo chroot /opt/evalx/runtimes/c-v11 /bin/bash -c "ldd /usr/bin/gcc | grep libm" 2>/dev/null

# Test if C++ runtime has standard library
echo "=== Testing C++ Library Loading ==="
sudo chroot /opt/evalx/runtimes/cpp-v11 /bin/bash -c "ldd /usr/bin/g++ | grep libstdc++" 2>/dev/null
```

```bash
# Test C compilation with math library
echo "=== Testing C Compilation in Chroot ==="
cat > /tmp/test_math.c << 'EOF'
#include <stdio.h>
#include <math.h>

int main() {
    printf("Square root of 16: %.2f\n", sqrt(16.0));
    printf("Power: %.2f\n", pow(2.0, 3.0));
    return 0;
}
EOF

sudo cp /tmp/test_math.c /opt/evalx/runtimes/c-v11/tmp/
sudo chroot /opt/evalx/runtimes/c-v11 /bin/bash -c "cd /tmp && gcc test_math.c -lm -o test_math && ./test_math" 2>/dev/null
```

 ```bash
# Create a test bubble sort program
cat > /tmp/bubble_test.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>

void bubbleSort(int arr[], int n) {
    for (int i = 0; i < n-1; i++) {
        for (int j = 0; j < n-i-1; j++) {
            if (arr[j] > arr[j+1]) {
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

int main() {
    int arr[] = {5, 2, 8, 1, 9, 3, 7, 4, 6};
    int n = 9;
    
    bubbleSort(arr, n);
    
    printf("Sorted: ");
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    return 0;
}
EOF

# Test compilation and execution in C runtime
echo "=== Testing Bubble Sort in C Runtime ==="
sudo cp /tmp/bubble_test.c /opt/evalx/runtimes/c-v11/tmp/
sudo chroot /opt/evalx/runtimes/c-v11 /bin/bash -c "cd /tmp && gcc bubble_test.c -o bubble_test && ./bubble_test" 2>/dev/null
```

```bash
# See what libraries are now available
echo "=== Library Inventory ==="
for runtime in /opt/evalx/runtimes/*; do
    if [ -d "$runtime" ]; then
        echo "--- $(basename $runtime) ---"
        sudo find $runtime/usr/lib/x86_64-linux-gnu -name "libm*" -o -name "libstdc++*" 2>/dev/null | head -3
        echo ""
    fi
done
```


```bash
echo "=== COMPREHENSIVE RUNTIME CHECK ==="
for runtime in /opt/evalx/runtimes/*; do
    if [ -d "$runtime" ]; then
        echo "🔍 $(basename $runtime):"
        
        # Check libm
        if sudo ls "$runtime/lib/x86_64-linux-gnu/libm.so"* > /dev/null 2>&1; then
            echo "   ✅ libm (math library) - FOUND"
        else
            echo "   ❌ libm - MISSING"
        fi
        
        # Check libstdc++ for C++
        if [[ "$runtime" == *"cpp"* ]]; then
            if sudo ls "$runtime/usr/lib/x86_64-linux-gnu/libstdc++"* > /dev/null 2>&1; then
                echo "   ✅ libstdc++ - FOUND" 
            else
                echo "   ❌ libstdc++ - MISSING"
            fi
        fi
        
        # Check usr/include
        if sudo ls "$runtime/usr/include/"* > /dev/null 2>&1; then
            echo "   ✅ Headers - FOUND"
        else
            echo "   ❌ Headers - MISSING"
        fi
        echo ""
    fi
done
```
