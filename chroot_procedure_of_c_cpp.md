## The REAL Solution for Future Languages:

**For ANY new language, just follow this pattern:**

```bash
# 1. Copy base chroot
sudo cp -r /opt/evalx/chroots/base /opt/evalx/chroots/NEW_LANGUAGE

# 2. Copy the main binary
sudo cp /path/to/language/binary /opt/evalx/chroots/NEW_LANGUAGE/usr/bin/

# 3. Copy ALL library dependencies
for lib in $(ldd /path/to/language/binary | grep -o '/[^ ]*' | grep -v '('); do
    sudo mkdir -p /opt/evalx/chroots/NEW_LANGUAGE$(dirname "$lib")
    sudo cp "$lib" "/opt/evalx/chroots/NEW_LANGUAGE$lib"
done
```

## Quick Fix for Right Now:

```bash
# Just fix the paths that exist
sudo mkdir -p /opt/evalx/chroots/java11/usr/bin
sudo cp /usr/lib/jvm/java-11-openjdk-amd64/bin/java /opt/evalx/chroots/java11/usr/bin/
sudo cp /usr/lib/jvm/java-11-openjdk-amd64/bin/javac /opt/evalx/chroots/java11/usr/bin/

# Use python3 instead of python3.9
sudo mkdir -p /opt/evalx/chroots/python/usr/bin
sudo cp /usr/bin/python3 /opt/evalx/chroots/python/usr/bin/
```

Then update your Python config to use `python3` instead of `python3.9`.

**The key insight:** Chroots are about copying existing system files, NOT installing new packages!
