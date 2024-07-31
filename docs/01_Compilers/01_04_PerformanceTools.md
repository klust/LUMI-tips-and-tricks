# Performance tools and metrics

## Performance counters

Partly based on messages from Orian in the chat:

-   Check which ones are available through PAPI: `papi_avail`.

-   Any L3 events are disabled as they are "uncore" events so that there is a security
    risk as they have to do with shared resources. 
    Uncore events require `/proc/sys/kernel/perf_event_paranoid` value to be set to 0 or -1
    which is not the case on LUMI.

-   There is a difference between counters on zen2 and zen3, so checking on the login nodes 
    does not show what you can use on the LUMI-C and LUMI-G compute nodes...
