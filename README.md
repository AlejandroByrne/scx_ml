# scx_ml

Development environment for sched-ext for UTNS.\
#### There are two git submodules present:
- One for storing a kernel version that is up to date with sched-ext functionality
- scx, the build tools for creating sched-ext schedulers. It points to Alejandro's fork, and it is synced with changes from main every now and then

### Development workflow explained
1) A Docker container is used to ensure consistent and functioning versions of the build tools
required to compile schedulers, run libbpf on them, and generate executables.
2) Virtme is used so that the schedulers can be run inside of reproducable system environments
in a fast manner. You can specify how many vCPUs are available, even limiting to 1, or restrict
memory, etc to test schedulers in controlled settings.
3) Because often your VM will not be as powerful as your bare-metal machine, the building/compiling
will be done on the bare-metal machine, and the code and executables will be placed in a shared
directory so that the VM can immediately access the newly compiled code without having to compile it
itself. 

#### Keep Dockerfile updated
Because the dependencies for scx development change sometimes, here are important versions to keep track of to ensure the Dockerfile is setting up a valid environment:
- Meson 1.2.0
- 

#### How to start the Docker container
To streamline your workflow of coding on your bare-metal machine, compiling within a Docker container, and running executables in a VM, you need to set up your Docker environment to share the `scx_ml` directory between your host machine and the Docker container. Here's how you can achieve this:

### **1. Build the Docker Image**

First, navigate to the directory containing your `Dockerfile` and build the Docker image.

```bash
cd ~/scx_ml/sched-ext-docker
docker build -t sched-ext-dev .
```

- **Explanation**: This command builds a Docker image named `sched-ext-dev` using the `Dockerfile` in the current directory.

### **2. Run the Docker Container with Volume Mounting**

Run the Docker container and mount the `scx_ml` directory so that it's accessible inside the container.

```bash
docker run -it --rm -v ~/scx_ml:/home/user/scx_ml sched-ext-dev
```

- **Options Explained**:
  - `-it`: Runs the container in interactive mode with a pseudo-TTY.
  - `--rm`: Automatically removes the container when it exits.
  - `-v ~/scx_ml:/home/user/scx_ml`: Mounts the `scx_ml` directory from your host into the container at `/home/user/scx_ml`.
  - `sched-ext-builder`: The name of the Docker image you built earlier.

### **3. Compile Your Code Inside the Container**

Once inside the container, navigate to the `scx` directory and compile your code.

### **4. Access the Compiled Executables in the VM**

Since `scx_ml` is shared between your bare-metal machine and the VM, you can now switch to your VM and run the newly generated executables.

### How to run virtme-ng VM for testing schedulers
1) cd into /linux
2) run ```vng -v --build --config ../sched-ext.config```
3) run ```make headers```
4) cd into /scx
5) run ```vng -vr ../linux```
6) Now, run schedulers with ```sudo ./build/scheds/c/scheduler_name```


### How to run virtme VM for testing schedulers

1) Go to the kernel source code and run these commands:
    - make defconfig
    - make -j$(nproc)
    - virtme-prep-kdir-mods
2) Then, run this command (doesn't matter what directory you run it in so long as --rwdir is specified)
    - virtme-run --kimg /home/ale/scx_ml/kernel/arch/x86/boot/bzImage --memory 8192 --rwdir /home/ale/scx_ml --qemu-opts -smp 4
    - another ```virtme-run --kimg /home/ale/scx_ml/kernel/arch/x86/boot/bzImage --memory 8192 --pwd```
    - virtme-run: This starts a minimal Linux environment using your custom kernel image (bzImage) in a temporary virtual machine.
    - --kimg /path/to/bzImage: Specifies the path to the kernel image (bzImage). This is the compiled Linux kernel you want to test, typically found in the arch/x86/boot directory of your kernel source tree after building it.
    - --memory 8192: Allocates 2048 MB (2 GB) of memory to the virtme environment. Adjust this value based on the available memory and your testing needs.
    - --pwd: Mounts your current working directory (on the host system) into the virtme environment as /dev/root. This allows you to access the files from the host's working directory directly inside virtme.
    - --rwdir /dev/code:/path/to/your/code: Mounts /path/to/your/code from the host as /dev/code inside virtme in read-write mode. This enables file sharing between the host and virtme without relying on the current working directory.d
    - --qemu-opts: allows passing additional arguments to configure the QEMU
    - -smp 4: An additional QEMU argument that specifies how many vCPUs to allocate to the VM.
        - By default, QEMU does not pin virtual CPUs (vCPUs) to specific physical CPUs (pCPUs). The vCPUs allocated to the VM can float across all available physical CPUs based on the host's scheduler. If you want to limit the VM to specific physical CPUs, you need to explicitly configure CPU affinity.
3) To kill the VM:
    - exit
    - pkill -f qemu-system

### How to reconfigure the meson build directory
```meson setup --reconfigure build -Dkernel_headers=../kernel/usr/include```

### Current task list
* Create a scheduler that uses linear regression infrerences asynchronously
    * Must create a method for collecting all relevant data pertaining to a task
    * Must create a pipeline for storing that data, running linear regression on it and storing the resulting lines of best fit for kernel space to infer from.