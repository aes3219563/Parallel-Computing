# Parallel-Computing
This test project is based on [Parallel Python](https://www.parallelpython.com/)<br>
They have stable version for Python2.x but the beta version for Python3 still have problems.<br>
I fix some of their files and put them in the pp folder. <br>
For pp workers:<br>
copy the whole pp folder to PATH_1, use cmd to run ppserver.py<br>
```python
python ppserver.py  -p 1888 -i your_ip -s "123456"
# open port 1888 at your parallel worker's ip. Password is 123456
```
For pp master:<br>
Also include your job script in the pp folder, here is an example to test the pp system<br>
```python
import math, sys, time, datetime
import pp
 
def isprime(n):
    """Returns True if n is prime and False otherwise"""
    if not isinstance(n, int):
        raise TypeError("argument passed to is_prime is not of 'int' type")
    if n < 2:
        return False
    if n == 2:
        return True
    max = int(math.ceil(math.sqrt(n)))
    i = 2
    while i <= max:
        if n % i == 0:
            return False
        i += 1
    return True
 
def sum_primes(n):
    """Calculates sum of all primes below given integer n"""
    return sum([x for x in range(2,n) if isprime(x)])
 
print ("""Usage: python sum_primes.py [ncpus]
    [ncpus] - the number of workers to run in parallel, 
    if omitted it will be set to the number of processors in the system
""")
 
# tuple of all parallel python servers to connect with
#ppservers = ()
ppservers = ("192.168.1.8:35000",)
#ppservers=("*",)
 
if len(sys.argv) > 1:
    ncpus = int(sys.argv[1])
    # Creates jobserver with ncpus workers
    job_server = pp.Server(ncpus, ppservers=ppservers, secret="123456")
else:
    # Creates jobserver with automatically detected number of workers
    job_server = pp.Server(ppservers=ppservers, secret="123456")
 
print ("Starting pp with", job_server.get_ncpus(), "workers")
 
# Submit a job of calulating sum_primes(100) for execution. 
# sum_primes - the function
# (100,) - tuple with arguments for sum_primes
# (isprime,) - tuple with functions on which function sum_primes depends
# ("math",) - tuple with module names which must be imported before sum_primes execution
# Execution starts as soon as one of the workers will become available
job1 = job_server.submit(sum_primes, (100,), (isprime,), ("math",))
 
# Retrieves the result calculated by job1
# The value of job1() is the same as sum_primes(100)
# If the job has not been finished yet, execution will wait here until result is available
result = job1()
 
print ("Sum of primes below 100 is", result)
 
start_time = time.time()
 
# The following submits 8 jobs and then retrieves the results
inputs = (500000, 500100, 500200, 500300, 500400, 500500, 500600, 500700, 500000, 500100, 500200, 500300, 500400, 500500, 500600, 500700)
#inputs = (1000000, 1000100, 1000200, 1000300, 1000400, 1000500, 1000600, 1000700)
jobs = [(input, job_server.submit(sum_primes,(input,), (isprime,), ("math",))) for input in inputs]
for input, job in jobs:
    print (datetime.datetime.now())
    print ("Sum of primes below", input, "is", job())
 
print ("Time elapsed: ", time.time() - start_time, "s")
job_server.print_stats()
```
