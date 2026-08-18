# Online Voting System in C

A console-based **Online Voting System** written in C. The project
manages an election, authenticates administrators, validates student
voter IDs, records votes, prevents duplicate voting, supports banning
voter IDs, deletes illegal votes, and stores election information in
text files.

## Features

-   Admin authentication
-   Create and configure a new election
-   Continue a previously saved election
-   Validate student voter IDs using:
    -   Year
    -   Branch code
    -   Roll number
-   Display available candidates
-   Cast votes using a voter ID
-   Prevent a voter from voting more than once
-   Ban specific voter IDs
-   Delete an illegal vote
-   Calculate the winner
-   Detect ties
-   Display complete election results
-   Display voting percentage
-   Store election and candidate information in text files

## Project Structure

The program is implemented as a single C source file and uses several
text files for persistent data.

### Important files

  -----------------------------------------------------------------------
  File                                Purpose
  ----------------------------------- -----------------------------------
  `ElectionInfo.txt`                  Stores election year, branch, total
                                      voters, and number of candidates

  `candidate1.txt`, `candidate2.txt`, Stores each candidate's vote count,
  ...                                 name, and voter roll numbers

  `Banned.txt` / `banned.txt`         Stores banned voter roll numbers

  `tmp.txt`                           Temporary file used while deleting
                                      an illegal vote
  -----------------------------------------------------------------------

> **Note:** The source code creates `Banned.txt` in `banID()`, while
> `loadElectionInfoFromFile()` attempts to open `banned.txt`. On
> case-sensitive systems, these are different filenames and should be
> made consistent.

## Voter ID Format

The program expects a 14-character student ID.

Example:

``` text
2018btecs00064
```

The program extracts:

-   **Year:** first 4 characters → `2018`
-   **Branch code:** characters 5--9 → `btecs`
-   **Roll number:** characters 10--14 → `00064`

A voter is considered valid when the extracted year and branch match the
current election settings and the roll number is within the configured
voter range.

## Main Components

### Data Structures

The program uses two main structures.

``` c
struct currentValidID {
    int year;
    char branch[6];
    int totalVoters;
};
```

This stores the valid voter-ID parameters for the current election.

``` c
typedef struct candidate {
    int cid;
    char cname[20];
    int votes;
} CANDIDATE;
```

This stores candidate information.

### Global Data

The program maintains:

-   `currentValidID` --- current election voter-ID rules
-   `candidateArray[20]` --- candidate information
-   `numberOfCandidates` --- number of candidates
-   `studentVotes[200]` --- voting status for students

## Important Functions

### `extractYear()`

Extracts the four-digit year from a voter ID.

### `extractRollNo()`

Extracts the five-digit roll number from a voter ID.

### `checkBranchCode()`

Checks whether the branch code in the voter ID matches the branch
configured for the election.

### `authenticateAdmin()`

Checks the administrator credentials.

The current source code expects:

``` text
Username: admin
Password: admin
```

### `initiateNewElection()`

Creates the in-memory configuration for a new election, including:

-   Election year
-   Branch code
-   Maximum roll number
-   Number of candidates
-   Candidate names

### `saveElectionInfoInFile()`

Stores the election configuration in `ElectionInfo.txt`.

### `createCandidateFiles()`

Creates a separate text file for each candidate and initializes its vote
count.

### `loadElectionInfoFromFile()`

Loads the saved election configuration, candidate information, votes,
and banned voter IDs from files.

### `isValid()`

Validates a voter ID by checking its length, year, branch, and roll
number.

### `isVoted()`

Checks whether a voter has already voted.

### `isBanned()`

Checks whether a voter ID has been banned.

### `saveVote()`

Records the vote in the appropriate candidate file, updates the
candidate's vote count, and marks the voter as having voted.

### `deleteIllegalVote()`

Removes a previously recorded vote and updates the candidate's vote
count and voter records.

### `getWinner()`

Finds the candidate with the highest vote count. It returns `-1` when a
tie is detected.

### `adminPanel()`

Provides administrator operations:

``` text
1. New Election
2. Continue Previous Election
3. Delete Illegal Vote
4. Ban User IDs
5. Result
6. Logout
```

### `studentPanel()`

Provides the voter interface. A student:

1.  Enters a voter ID.
2.  Is checked for validity.
3.  Is checked for ban status.
4.  Is checked for previous voting.
5.  Selects a candidate.
6.  Casts the vote.

## How the Program Works

``` text
                Start
                  |
          +-------+-------+
          |               |
       Admin           Student
          |               |
    Authentication     Enter ID
          |               |
    Admin Panel       Validate ID
          |               |
    Election Setup    Check Ban
          |               |
    Candidate Files   Check Already Voted
          |               |
    Voting Management Select Candidate
          |               |
    Results / Ban /    Save Vote
    Delete Vote            |
          |               |
          +-------+-------+
                  |
                 End
```

## File-Based Storage

The application uses standard C file handling functions such as:

``` c
fopen()
fclose()
fprintf()
fscanf()
fseek()
remove()
```

This allows election data to persist between program runs.

## Requirements

The source uses:

``` c
#include <stdio.h>
#include <conio.h>
#include <string.h>
#include <stdlib.h>
```

Because the program uses `conio.h` and `getch()`, it is intended for a
compiler/environment that supports these functions, commonly older
Windows-oriented C environments.

## How to Run

1.  Save the source code as a `.c` file, for example:

``` text
voting_system.c
```

2.  Compile it using a C compiler that supports the required
    headers/functions.

3.  Run the executable.

4.  Configure an election through the administrator panel before
    students cast votes.

## Typical Workflow

### Administrator

``` text
Login
  ↓
New Election
  ↓
Enter year
  ↓
Enter branch
  ↓
Enter maximum roll number
  ↓
Enter number of candidates
  ↓
Enter candidate names
  ↓
Election files created
```

### Student

``` text
Enter voter ID
  ↓
Validate ID
  ↓
Check banned status
  ↓
Check whether already voted
  ↓
Select candidate
  ↓
Vote recorded
```

### Results

The administrator can view:

-   Winner
-   Tie status
-   Votes received by every candidate
-   Overall voting percentage

## Security and Production Considerations

This project is suitable as a **C programming / file-handling project or
educational demonstration**, but it should not be treated as a secure
real-world election system without substantial redesign.

Important areas that would need improvement include:

-   Hard-coded administrator credentials
-   Plain-text election data
-   Lack of strong authentication
-   No encryption
-   File-based vote storage
-   Limited input validation
-   Fixed-size arrays
-   Reliance on `getch()`
-   Potential file-handling and boundary issues
-   No protection against concurrent access
-   No cryptographic verification of votes

## Known Source-Code Considerations

The provided implementation contains some areas that should be reviewed
before production use:

1.  `Banned.txt` and `banned.txt` use different capitalization.
2.  Several `fopen()` failure branches call `fclose()` even when the
    file pointer is `NULL`.
3.  Input sizes are not consistently bounded with `scanf("%s", ...)`.
4.  The program uses fixed-size arrays:
    -   Maximum 20 candidates
    -   Maximum 200 voter records
5.  File-reading loops using `while(!feof(...))` can process an
    unsuccessful read.
6.  Administrator credentials are hard-coded.
7.  Vote and voter information is stored in plain text files.
8.  Candidate names are read as single whitespace-delimited strings.

These are useful areas for future improvements and debugging.

## Future Improvements

Possible upgrades include:

-   Better input validation
-   Password hashing
-   Secure administrator authentication
-   Database-based storage
-   Encryption
-   Role-based access
-   Better duplicate-vote protection
-   Improved file handling
-   Dynamic data structures
-   Detailed audit logs
-   Exportable election reports
-   Improved user interface
-   Web or mobile frontend
-   Secure API/backend architecture

## Technologies Used

-   **Language:** C
-   **Libraries:** `stdio.h`, `conio.h`, `string.h`, `stdlib.h`
-   **Storage:** Text files
-   **Concepts:** Structures, arrays, functions, strings, file handling,
    authentication, validation, voting logic

## License

No license is specified in the provided source code. Add an appropriate
license if this project is intended to be distributed publicly.
