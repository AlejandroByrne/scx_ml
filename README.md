# scx_ml

Development environment for sched-ext for UTNS.\
There are two submodules present:
- One for storing a kernel version that is up to date with sched-ext functionality
- scx, teh build tools for creating sched-ext schedulers. It points to Alejandro's fork, and it is synced with changes from main every now and then


### Current task list
* Create a scheduler that uses linear regression infrerences asynchronously
    * Must create a method for collecting all relevant data pertaining to a task
    * Must create a pipeline for storing that data, running linear regression on it and storing the resulting lines of best fit for kernel space to infer from.