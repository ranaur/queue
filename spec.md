Bash command for managing job queues.

A queue has a name, every job added to it will be executed sequentially. 

A queue is modelled a subdirectory in /var/lib/queues/<queque-name>. 

In this directory will go:
    1) the config file for the queue: config
    2) the lock file (that exists when soem command is making changes on the queue): .pidlock
    3) the jobs that will be executed. Theese files contains a buch on variables taht will define the job:
        ID
        CREATION_DATE
        COMMAND
        STARTING_DIRECTORY

The permissions (read/write) of the directory will define who will be able add new jobs. The jobs will be executed by the owner of teh directory.

The directory /var/log/queues will also have a subdirectory for each queue that will hols the log output of the executed job. Also will have a file, queue.log that will hold the genreal logs for the queue handling.

A queue can have the following states:
    normal/paused => if paused, new jobs only queue but are not executed
    empty/non-empty => empty if there is no jobs

Log Output Format:

A textfile where each line is prefixed by:
    job: job number/id
    cmd: job that was executed
    dir: Startnig path (cwd)
    bgn: Begin date/time
    out: stdout lines
    err: stderr lines
    end: End date/time
    dur: duration
    ret: return code

General Log format is:

yyyyMMdd hh:mm:ss.ssss;command runned in queue;message

The directory /var/run/queues will hold a file <queue_name>.pid that will hold the pid of the running job.

Command Syntax:

queue queue_name command [parâmers ...]

Commands:

new/setup - creates a new queue.
    creates the directories
    initializes the first log entry of teh general queue log (new: ...)
    create default config file in 

destroy = removes a queue. If the queue is non-empty, shows an error message, unless -f parameter is passed.
    erases the directories for the queue in /var/lib/queues
    Parameters:
    --force/-f => destroys even if there is jobs in the queue or runnig jobs.
    --log/-l => erases log directory /var/log/queues/<queue_name> too

up = adds the comamnd to the queue. Normally the command will be the executed after every other job already in the queue (unless -j is passed)
    queue queue_name [queue parameters] up command [command parameters]

    Parameters:
    --starting-dir/-d directory => cd to this directory before running the command (default: current dir)
    --jump/-j => make the command to be the next to be executed (jump queue)

wait = waits iuntil the queue finish the last job

pause = pause the queue
    parameters:
        --wait/-w => waits until the current running job finishes (is tehre is one)

resume = resume a pause, running the next command in queue, if any

status = show the queue status
    Show: queue name, state (paused, normal), number of jobs ququed

list = list all jobs waiting in queue (in order)
    job sequence: command
    
    Parameter:

    -i/--id = shows job id instead the sequence number

clear = remove all jobs from the queue

remove = remove a job from the queue

kill = emits a kill signal for the running job
    passes all the parameters for the kill(1) command

log = shows the queue's general log
    Parameter:
    -t/--tail <n>
    -s/--since date
    -f/--follow - similar to tail-f

out = shows the running job output log
    Parameter:
    -t/--tail <n>
    -s/--since date
    -f/--follow - similar to tail -f -w <pid of the job>

configure = opens the editor to change the queue config file:

Lines in the config file and meaning:
WAIT_TIME=<number> # number of seconds to wait between jobs <default: 5>
ON_ERROR=pause/continue # how to do when command returns not 0: pause queue, continue to the next  <default: continue>
RETRY_WAIT=<number> # number of seconts to wait for retry
MAX_RETRIES=<number> # maximum number of reties

deamon = runs as deamon (detached) watching every existing queue ans stringing it if it has job to execute.

