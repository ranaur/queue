queue is Bash command for managing queues.

A queue is a queue (normally FIFO) of executions that are run in sequence. At any time you can add a comand to the queue. It will run next if the queue is empty, or wait until all the comands in the queue finished to start.

There is a number of commands you can use to handle the queue:

# Command: new/setup

    queue new [options] QUEUE_NAME

Creates a new queue named "QUEUE_NAME"

Options:

-g/--global - creates a globa (e.g. not local) queue
-d/--default - sets the new queue as default
-e/--edit - edits the new queue config file
-q/--queue-file - uses the file as the config file for the queue

Returns:
    0 if queue is created
    -1 if any error occourred

# Command: destroy

    queue destroy QUEUE_NAME [options]

Removes all data for the queue. The queue must be idle.

Options:

-f/--force - destroys even quques with pending commands. If there is a running command pauses the queue, kills -9 the running job, wait it to end, and the removes the queue.

# Command: up

    queue [global options] up [options] command [command options ...]

Queues up the command. When ready, it will run "command [command options ...]". Ready means:
    runs immediately if the queue is empty and ready.
    waits until the last queued command run to start this command

Options:

-j/--jump - put the comand as the next command in the queue to run, not the last (default).
-d/--directory - uses anternative directory as run directory. (default: current directory)

# Command: set

    queue [global options] config variable value

Changes a config variable in the .queuerc file. Existing variables:

    default-queue - default queue to execute if no -q parameter is passed (default: default)

# Command: list

    queue [global options] list [options]

List all the commands of the queue. Return in the format:

SEQ Status Command

Where: SEQ is the sequence number of the job (up to 6 numbers), Status is the status of the commands [W = WAITING, R = RUNNING, F = FINISHED, E = ERROR/FAILED], command is the command that will run

Options:

-d/--directories - list the working directories after the command (wd: directory)
-p/--pending - list only waiting our running commands
-x/--executed - list only finished or failed commands
-w/--waiting - list only waiting commands
-r/--running - list only running commands
-f/--finished - list only finished commands
-e/--error/--failed - list only failed commands
-a/-all - list the name of all existing queues, prefixed with USER: ou GLOBAL:
-u/--user - list all user queues
-g/--global - list all global queues

# Command: status

    queue [global options] status [options]

Return information about the queue. Returns:

State of the queue: idle, running, paused
Number of queued jobs: number of jobs wainting to run
Number of finished jobs: number of jobs that ended with return code 0
NUmber of failed jobs: number of jobs that ended with return code != 0
Sequence number of the job currently running: SEQ, pid and command line.

# Command: command

    queue [global options] command/cmd [SEQUENCE_NUMBER] subcommand [subcomamnd parameters]

Acts or edits a command. The default is to act in the running comand or first command if the queue is paused. Passing a SEQUENCE_NUMBER changes this behaviour for the command SEQUENCE_NUMBER.

Allowed subcommands:

    remove/delete - removes command from the queue
    kill [-signal] - sends a kill signal to a running command
    movelast - move the commant to the end of the queue
    movefirst - move the command to the start of the queue (to be executed right after the current runnig command)
    edit - edit command descriptor (opens editor for the command descriptor file)
    out = shows the job output log (if exists)
        Parameter:
        -t/--tail <n> - shows only the last N lines
        -f/--follow - similar to tail -f -w <pid of the job>

# Global options:

-q/--queue QUEUE_NAME - use QUEUE_NAME as operating queue instead of the default

A queue name must be case insensitive, and have no / in them and cannot start with a period ".".

A job ID is always a 6 digit number from 000000 to 999999.

The queue can be global (all users) or local (only current user). When is global the queue directory is in /var/lib/queue. When local, it is in ~/.queue.

In the queue directory there is:

config - a file with the general options for quque handling
queue_name/ - a directory for each existing queue.

Each queue directory can have the following diretories:

queue.config - config for the queue
queue.log - queue log
SEQUENCE_NUM.pid - pidfile of the running command. Removed when command ends.
SEQUENCE_NUM.descriptor - file with the command information
SEQUENCE_NUM.log - log file with the command output (see bellow)
SEQUENCE_NUM.env - environment file for the command (optional)
lockfile - eventualy a logfile with the PID of the process changing the directory

quueconfig has the following variables:

ON_FAILURE=default behaviour when the command returns != 0: skip (jumps to the next), pause (pause the queue)
MAX_RETRY=maximum number of retries when the command fails (default:0 or no retry)
WAIT=number of seconds to wait before starting each command
STATE=paused/normal

The command descriptions has the following variables:
COMMAND=command to execute with parameters
DIRECTORY=starting directory for the command

When run by root, the job will be executed by the owner of the directory. 
the permissions (read/write) of the directory will define who will be able add new jobs, this is specially importatr in global jobs. The jobs will be executed by the owner of the directory.

Lines in the global config file and meaning:

WAIT_TIME=<number> # number of seconds to wait between jobs <default: 5>
ON_ERROR=pause/continue # how to do when command returns not 0: pause queue, continue to the next  <default: continue>
RETRY_WAIT=<number> # number of seconts to wait for retry
MAX_RETRIES=<number> # maximum number of reties



Command Log Output Format:

A textfile where each line is prefixed by:
    seq: job sequence number
    cmd: job that was executed
    dir: Startnig path (cwd)
    bgn: Begin date/time
    out: stdout lines (0 to many)
    err: stderr lines (0 to many)
    end: End date/time
    dur: duration
    ret: return code
    kll: number of the signal of the command kill 
if the job was interurpted/killed, return code will never be 1

General queue Log format is:

yyyyMMdd hh:mm:ss.ssss;command runned in queue;message

##########################################

Command Syntax:

queue queue_name command [parâmers ...]

# Other Commands:

wait = waits until the queue finish the last job

pause = pause the queue
    parameters:
        --wait/-w => waits until the current running job finishes (is there is one)

resume = resume a pause, running the next command in queue, if any

clear = remove all jobs from the queue
    -p/--pending - clear only waiting commands
    -x/--executed - clear only finished or failed commands
    -w/--waiting - clear only waiting commands
    -f/--finished - clear only finished commands
    -e/--error/--failed - clear only failed commands

log = shows the queue's general log
    Parameter:
    -t/--tail <n> - shows last n lines
    -s/--since date - shows only after date
    -f/--follow - similar to tail-f

configure = opens the editor to change the queue config file

deamon = runs as deamon (detached) watching every existing queue ans stringing it if it has job to execute.

