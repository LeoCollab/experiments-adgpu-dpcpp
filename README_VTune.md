# README for VTune Profiling

These guidelines are based on AutoDock-GPU's (DPC++ version) commit `6e2683f`. Hence, it requires fixes and extra flags for VTune profiling.

Below you can find AutoDock-GPU code modifications as well as compilation & profiling commands for both OpenCL and DPC++ versions.

## Code Changes

Keep in mind that in the meantime, **some or all of these code changes might have been already merged into the mainline code.**


```
u158538@login-2:~/AutoDock-GPU$ git diff

diff --git a/Makefile.OpenCL b/Makefile.OpenCL
index 7b355c3..13fe96f 100644
--- a/Makefile.OpenCL
+++ b/Makefile.OpenCL
@@ -162,7 +162,8 @@ OCL_DEBUG_ALL += -DCMD_QUEUE_OUTORDER_ENABLE
 endif
 
 ifeq ($(CONFIG),FDEBUG)
-       OPT =-O0 -g3 -Wall $(OCL_DEBUG_ALL) -DDOCK_DEBUG
+       OPT =-O0 -g3 -Wall 
+#$(OCL_DEBUG_ALL) -DDOCK_DEBUG
 else ifeq ($(CONFIG),LDEBUG)
        OPT =-O0 -g3 -Wall $(OCL_DEBUG_BASIC)
 else ifeq ($(CONFIG),RELEASE)


diff --git a/Makefile.dpcpp b/Makefile.dpcpp
index ba3b917..7d0fd37 100644
--- a/Makefile.dpcpp
+++ b/Makefile.dpcpp
@@ -99,9 +99,10 @@ CONFIG=RELEASE
 #CONFIG=FDEBUG
 
 ifeq ($(CONFIG),FDEBUG)
-       OPT =-O0 -g3 -Wall -DDOCK_DEBUG
+#      OPT =-O0 -g3 -Wall -DDOCK_DEBUG
+       OPT =-g3 -Wall #-DDOCK_DEBUG
 # for vtune
-#      OPT +=-gline-tables-only -fdebug-info-for-profiling
+       OPT +=-gline-tables-only -fdebug-info-for-profiling
 ifeq ($(DEVICE), GPU)



diff --git a/Makefile.OpenCL b/Makefile.OpenCL
index 7b355c3..13fe96f 100644
--- a/Makefile.OpenCL
+++ b/Makefile.OpenCL
@@ -162,7 +162,8 @@ OCL_DEBUG_ALL += -DCMD_QUEUE_OUTORDER_ENABLE
 endif
 
 ifeq ($(CONFIG),FDEBUG)
-       OPT =-O0 -g3 -Wall $(OCL_DEBUG_ALL) -DDOCK_DEBUG
+       OPT =-O0 -g3 -Wall 
+#$(OCL_DEBUG_ALL) -DDOCK_DEBUG
 else ifeq ($(CONFIG),LDEBUG)
        OPT =-O0 -g3 -Wall $(OCL_DEBUG_BASIC)
 else ifeq ($(CONFIG),RELEASE)


diff --git a/Makefile.dpcpp b/Makefile.dpcpp
index ba3b917..7d0fd37 100644
--- a/Makefile.dpcpp
+++ b/Makefile.dpcpp
@@ -99,9 +99,10 @@ CONFIG=RELEASE
 #CONFIG=FDEBUG
 
 ifeq ($(CONFIG),FDEBUG)
-       OPT =-O0 -g3 -Wall -DDOCK_DEBUG
+#      OPT =-O0 -g3 -Wall -DDOCK_DEBUG
+       OPT =-g3 -Wall #-DDOCK_DEBUG
 # for vtune
-#      OPT +=-gline-tables-only -fdebug-info-for-profiling
+       OPT +=-gline-tables-only -fdebug-info-for-profiling
 ifeq ($(DEVICE), GPU)
 # for AOT compile and debug
 #      OPT+=-fsycl-targets=spir64_gen-unknown-unknown-sycldevice -Xs "-device ats -internal_options -cl-kernel-debug-enable -options -cl-opt-disable"


diff --git a/host/src/performdocking.cpp.OpenCL b/host/src/performdocking.cpp.OpenCL
index 787c880..ba227c0 100644
--- a/host/src/performdocking.cpp.OpenCL
+++ b/host/src/performdocking.cpp.OpenCL
@@ -84,7 +84,7 @@ Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
 #ifdef __APPLE__
        #define KGDB_GPU " -g -cl-opt-disable "
 #else
-       #define KGDB_GPU " -g -O0 -Werror -cl-opt-disable "
+       #define KGDB_GPU " -g -cl-opt-disable "
 #endif
 #define KGDB_CPU " -g3 -Werror -cl-opt-disable "
 // Might work in some (Intel) devices " -g -s " KRNL_FILE


diff --git a/host/src/performdocking.cpp.dpcpp b/host/src/performdocking.cpp.dpcpp
index cdf501f..eb03a42 100644
--- a/host/src/performdocking.cpp.dpcpp
+++ b/host/src/performdocking.cpp.dpcpp
@@ -1273,11 +1273,11 @@ int docking_with_gpu(const Gridinfo *mygrid, Dockpars *mypars,
        #ifdef DOCK_DEBUG
                para_printf("\nExecution starts:\n\n");
                para_printf("%-25s", "\tK_INIT");fflush(stdout);
-               cudaDeviceSynchronize();
+//             cudaDeviceSynchronize();
        #endif
        gpu_calc_initpop(kernel1_gxsize, kernel1_lxsize, pMem_conformations_current, pMem_energies_current);
        #ifdef DOCK_DEBUG
-               cudaDeviceSynchronize();
+//             cudaDeviceSynchronize();
                para_printf("%15s" ," ... Finished\n");fflush(stdout);
        #endif
        // End of Kernel1
@@ -1288,7 +1288,7 @@ int docking_with_gpu(const Gridinfo *mygrid, Dockpars *mypars,
        #endif
        gpu_sum_evals(kernel2_gxsize, kernel2_lxsize);
        #ifdef DOCK_DEBUG
-               cudaDeviceSynchronize();
+//             cudaDeviceSynchronize();
                para_printf("%15s" ," ... Finished\n");fflush(stdout);
        #endif
        // End of Kernel2
(END)
```


## Compile

```
$ make DEVICE=OCLGPU NUMWI=64 CONFIG=FDEBUG
```

```
$ make DEVICE=XeGPU NUMWI=64 CONFIG=FDEBUG_VTUNE
```

##  Profile

```
$ ./run_vtune_profiling.sh
```

