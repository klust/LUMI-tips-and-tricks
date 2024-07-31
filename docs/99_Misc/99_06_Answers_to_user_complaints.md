# Answers to frequent user complaints

## You should have warned that my data is removed 90 days after project termination

-   All users get an email the day before the project ends, sent from `idmadm@csc.fi`,
    with subject: "Warning:LUMI project 45YXXXXXX end-date is approaching" 

    ```
    Dear LUMI User,

    This message is sent to all members of the project 46YXXXXXX (<NAME OF THE PROJECT>).

    Your LUMI project 46YXXXXXX is ending at <DATE>>.

    The end of the project will lead to termination of all resources (allocation) in the project. 
    You will be able to login and access your data. 
    The project data will be stored and available in the LUMI cluster for 90 days after the project
    losure. After 90 days any data left in the LUMI cluster will be deleted. It is recommended to 
    transfer and remove the data from LUMI before the end of the 90 days grace period.
    If you are the Project Principal Investigator (PI), please assure that all the project members 
    are aware about the approaching end date. Project PI can request a project extension. Please 
    contact your national Resource Allocator for the extension request.
    ```

    (There is a second mail when the project is actually fully closed but that doesn't help.)

-   Agreed that we do not explicitly warn for the 90 days after project end date policy in the
    documentation, but the 
    [tables on the "Data storage options" page](https://docs.lumi-supercomputer.eu/storage/#lumi-network-file-system-disk-storage-areas)
    do clearly say that there is no backup which already makes it clear that users should maintain
    their own backup of precious data outside LUMI.

-   We do warn that LUMI is not a long-term data storage option in our introductory courses.

-   The fact that data use is billed should also make clear that that data will not stay forever
    on the system when the project that can be billed for the data use has ended.

-   TODO: Add this to the rotating part of the message-of-the-day.
