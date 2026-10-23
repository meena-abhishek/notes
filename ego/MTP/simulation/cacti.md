The time delay between **activating a row** and **reading/writing a column** within that row.
The memory controller must wait exactly **tRCD** clock cycles for the data to stabilize in the sense amplifiers before it is allowed to send the `READ` or `WRITE` (CAS) command to pull the specific data bits out.

The time required to **close an open row** and prepare the memory bank to open a completely new row.
The memory controller must wait exactly **tRP** clock cycles after issuing the precharge command before it is allowed to send a new `ACTIVATE` command to a different row.

