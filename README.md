# 3-rtos- methods for synchronizing threads of different processes
Functions that are involved in creating a mutex in shared memory:
Shm_open() – creates a file that describes a mutex object
Mmap() recommends mapping the memory for the mutex
Pthread_mutexattr_setpshared() - is responsible for setting the attributes of the shared mutex
Pthread_mutex init() – mutex initialization
Code snippets as where to use the above functions:

/* Shared memory allocating for the mutex object*/

/*_____*/

mutex=(pthread_mutex_t*) mmap (NULL, sizeof(pthread_mutex_t), PROT_READ|PROT_WRITE,MAP_SHARED,des_mutex,0);

if (mutex == MAP_FAILED )

{

perror("Error on mmap on mutex\n");

exit(1);

}

/*Creating file descriptor of the shared structure*/

/*___*/

des_data=shm_open(DATA_MES,O_CREAT|O_RDWR|O_TRUNC,mode);

if (des_data < 0){

perror("failure on shm_open on des_mutex");

exit(1);

}

if (ftruncate(des_data, sizeof(struct data_str_t)) == -1){

perror("Error on ftruncate to sizeof struct data_str_t\n");

exit(-1);

}

/* Shared memory allocating for the shared structure*/

/*____*/

mydata=(struct data_str_t*)mmap(NULL,sizeof(struct data_str_t),PROT_READ | PROT_WRITE,MAP_SHARED,des_data,0);

if (mydata == MAP_FAILED ){

perror("Error on mmap on mydata\n");

exit(1);

}
/* set mutex shared between processes */

pthread_mutexattr_t mutexAttr;

pthread_mutexattr_setpshared(&mutexAttr, PTHREAD_PROCESS_SHARED);/*____*/

pthread_mutex_init(mutex, &mutexAttr);

## 4.
Unlocking steps to run the program:
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/69d78628-e568-4533-a2a2-392b307b47e4)

a) After running, the variables are different from each other: var1 is not equal to var2
This means that they do not work properly, both running processes report a lack of "interoperability" - different values.
The result of the action
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/2c684bbc-0255-4855-b8b3-0e89bf0233e6)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/db45f9ca-a8c9-4fde-a095-d6ce06ef81fd)

The result of the MUTEX PROCES 1 system:
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/9656022e-cea7-4fdc-bea9-7440cd79fdd8)

b) The process scheduling algorithm is Round-Robin RR operating with priority 20 in the code, marked in the system (in the console) with the value 60. [checked after starting the process using the ps -e command where the PID number was checked and then the ps - cT command – p and adding the previously found pid number]

![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/61666b73-b4fd-4a4e-a8b8-4f14e25af1ef)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/373489b6-209f-4fa9-8827-f40e305269c8)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/0157860a-f52d-493e-b2f6-49a7659cea5d)

RESULT OF OPERATION OF THE MUTEX PROCESS 2 SYSTEM:
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/3ddf7c2a-768c-44fa-b3df-6f05c97d7e01)

a) The process scheduling algorithm is Round-Robin RR, operating with priority 20 in the code, and marked in the system (in the console) with the value 60. [checked as above]
Both MutexProcess 1 and Mutex Process 2 have the same scheduling algorithms and priority.
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/5a8c6ed3-4429-4c52-8979-022cba22195c)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/d887874c-c69b-41cd-9d1d-8e0d8755349d)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/e13c6fd0-c07b-4cb9-9e1f-4032e3e5a931)

b) The operation of var1, var2, var3 is of this type because there is no mutex blocking in critical sections, so threads executing critical sections of the code collide with each other "competition for shared resources"
The reason may also be the type of algorithm used, which has the nature of competitive behavior of threads (var 1 and var 2) for processor time. For example, one thread may lose CPU time while incrementing among itself.

d) The solution to the problem may be to use thread synchronization methods, e.g.:
pthread_mutex_lock(mutex) at the beginning of a critical section to lock it and 
pthread_mutex_unlock(mutex) to unlock it after a critical section.

## 5.
a) Attributes that indicate that the mutex can be shared between threads of different processes are setting the PTHREAD_PROCESS_SHARED flag and defining the mutex in shared memory. The mutex Attr attribute is shared between processes (set mutex shares between processes): pthread_mutexattr_t mutexAttr; pthread_mutexattr_setpshares(&mutexattr, PTHREAD_PROCESS_SHARED); pthread_mutex_init(mutex,&mutexAttr);

b) After supplementing the update_thread function with mutex elements, the programs work correctly. Result: Variables var1 and var2 are equal.

c) The my data structure is shared in memory for both processes. Processes see the data structure in the same way, that is, through exclusive access to a critical section of the code.
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/a3526f57-4072-400b-a4db-293c002dd5d3)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/2d9842a1-b91f-4129-b8c2-20e9fe4b312f)

## 6.
This is how the states change in a predetermined sequence
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/cb09a10f-87ae-4b9c-9551-8dd6a8f1febe)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/7b2b1d73-a3b8-44f4-a90f-4de78852cd7c)

After starting the programs supplemented with appropriate functions, in the CondvarProcess_1 program the status changes from 0 to ->1 and in CondvarProcess_2 from 1 to ->0. The sequences of changes are repeated, which is as intended

## 7 
a) the result of changing program states: CondvarProcess_1 changes from 0 to ->1 and CondvarProces_2 changes from 1 to ->2 and from 2 to ->0. The upper part of the photo is a running program and the lower part is a blocked process - it returns only one sequence.
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/fb908b48-3613-4b3d-8fb8-5067f182bf67)

result from eclipse console:
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/e7429797-98d9-4491-8c81-4cb1e3b944b2)

b) The program does not work properly because it blocks after changing state 1 to state 2
c) The solution to the problem was to replace pthread_cond_signal with pthread_cond_broadcast in thread states. Commands are used to signal a change in a conditional variable so that the information reaches all threads and they can react appropriately.

## 8.
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/901706e1-97f4-45a8-a3c9-4963924a12ef)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/79b9e6cf-01f3-4c36-8734-91c2ccc72b5b)

a) The program after modification works correctly. CondvarProcess_1 changes the state from 0->1 or 3->0, while CondvarProcess_2, depending on the internal structure of variables, changes the state from 1->2 or from 2 ->0 and from 1->3
b) The solution to the problem was to replace pthread_cond_signal with pthread_cond_broadcast in thread states (changing the signaling of the condition variable)

## 9.*
a)Programs use an unnamed semaphore. The commented name in the code is //#define Named
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/40691bf6-896b-4a95-889f-2eb6ce6c6b85)

b) To create a semaphore used in these programs, you must register the name of the shared memory area that will be assigned to a given semaphore. First, you create a descriptor file for the unnamed semaphore, then share the memory containing the unnamed semaphore, and then use sem_init(mySemaphore,1,0); the semaphore is initialized.

## 10.
SemProc1 runs as a 32764 producer, increments the counter on creation and sends the semaphore(s). SemProc 2 acts as a receiver of semaphores for subsequent consumers from individual producers while the program is running.
Both programs are dependent on each other and work only when launched simultaneously (Starting should start with the producer SemProc1 and then the consumer SemProc 2. When running only 1 of them, they do not work.)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/2486ee00-a528-443d-ba14-5ec698e3170e)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/01df0e26-6975-4fea-bed0-5e4b915077bb)

## 11*
a) Programs use a named semaphore # define Named (sem.–abbreviation semaphore identifier pointer to memory structure)
name sem.Semex
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/7be2a9f1-bc22-4d0c-93c6-102bf7dbceb6)

b)Creating this type of semaphore involves registering the semaphore name in the namespace and starts by creating sem_open and registering the semaphore name in the files on disk. Resources are then allocated for this semaphore, and using sem_init(mySemaphore,1,0); the semaphore is initialized.

## 12.
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/0b2e48d7-15d0-4aae-8100-ed53d45f968a)
![image](https://github.com/AsiaEwa/3-rtos/assets/101841759/57988d44-a316-48ca-af0c-a97e15565f41)
a) Operation: NamedSem1 as producer 0 increments the counter by sending a semaphore, and NamedSem2 acts as a recipient of semaphores from the producer to consumers.
B) the path to the name of the named semaphore is cd /dev/shm 

## *addition to 9 & 11 :
UNNAMED SEMAPHORS: are designed to synchronize threads within one process. An unnamed semaphore is accessed via its address. It can also be used to synchronize processes as long as it is placed in shared memory.
An unnamed semaphore is accessed via the semaphore address. Hence the name unnamed semaphore.
Before a semaphore can be used, it must be declared as an object of type sem_t and the memory used by the semaphore must be explicitly allocated to it. If the semaphore is to be used in different processes, it should be placed in previously allocated shared memory.
1. Create a memory segment using the shm_open function.
2. Specify the segment dimension using the ftrunc function.
3. Map the shared memory area in the process data space - mmap. Semaphore initialization - sem_init function

Before use, the semaphore should be initialized. int sem_init(sem_t *sem, int pshared, unsigned value)

(pshared: When the value is non-zero, the semaphore can be placed in shared memory and accessed by multiple processes; value The initial value of the semaphore)

Waiting on a semaphore – sem_wait sem_wait(S) function. int sem_timedwait(sem_t *sem, struc timespec timeout)
Signaling on the semaphore – sem_post function: int sem_post(sem_t *sem)
Deleting a semaphore: int sem_destroy(sem_t *sem)

SEMAPHORES NAMED:
Named semaphores are accessed by name. This type of semaphore is more suitable for synchronizing processes than threads. Unnamed semaphores run faster than named ones. Named semaphores are identified in processes by their name. A named semaphore is operated in the same way as an unnamed semaphore, except for the functions of opening and closing the semaphore. 

Opening a semaphore - sem_open function To use a named semaphore, access it via the function: sem_t *sem_open(const char *sem_name, int oflags, [int mode, int value]) sem_name The name of the semaphore should start with the "/" character.

offlags Create and open mode flags: O_RDONLY, O_RDWR, O_WRONLY. When the semaphore is created, use the O_CREAT flag
Closing a named semaphore - function sem_close int sem_close( sem_t * sem) The function releases the resources of the semaphore and unblocks processes that are blocked on it.

semaphore operations in the POSIX standard:
* sem_open - creating a named semaphore
* sem_init - emaphore initialization
* sem_wait - waiting at the semaphore
* sem_trywait - downloading the resource
* sem_timewait - waiting to expire
* sem_post - signaling
* sem_close - closing the semaphore
* sem_unlink - semaphore deletion

  
