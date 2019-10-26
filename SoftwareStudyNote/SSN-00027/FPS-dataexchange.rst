******
Trade Study: Cobra operational data (spot measurement, target) handling on what way: MHS, DB, and so on
******

Abstract
******

This is a trade study on how FPS and modules under control to handle data 
of Cobra operational data, such as spot measurement results, current target 
positions, or commanded target positions, over inter-module communication. 
As PFS ICS infrastructure, PFS provides status exchange with automatical 
archive over MHS, and also shared database system. 

Each set of Cobra operational data exchanged among FPS and modules under 
control will be a several sets of ~2400 or ~2500 (for sets with sports of 
fixed fiducials) data points, which could be 10kB (4byte integer or single 
floating point) or 20kB (8byte integer or double floating point) data volume 
per set, and several hundled kB in total for whole to be exchanged.
This is relatively large amount of data compared to other status exchange 
which contains just several data points in text, and is better to consider 
of its handling to exchange among actors. Also keeping and easy referring of 
these data could be a key for smooth operation of performance verification 
and motor map regeneration of Cobra at offline, and that means easily 
accessible as dataset is important.

Technical Details
******

Backgrounds
======

We had wide variety of discussion on how we will handle and pass values among 
control modules for Cobra configuration, such as IIC, FPS (former PFICS) with 
m2t+MPS, MCS, and ETS. Several ways such as set array/dict of values in 
KeysDictionary and send as normal way, passing data as one text command 
option with serialized, or having files (e.g. FITS) for data exchange.
Although keeping overhead time for commanding and data transfer as minimum 
is a key for performance of Cobra configuration, process time taken for 
conversion from/to python native data for message exchange is well quicker 
than hardware movement, and also these conversion only happens once (or one 
pair of from and to) per one command flow but not in large multiplexity for 
processing time. One note from architecture point of view on the tron system, 
all data transfer both command (or command option) and status are exchanged 
only by text (mostly us-ascii, but text in like UTF-8 could be possible?) 
but not binary, this means even script passes array/dict of values to the 
client library of the tron, serialization process and its cost on processing 
time are applied.

In parallel to command and status flow, keeping history of measured 
statistics for spots of back-illuminated fibers by MCS, commanded positions 
of current and target from FPS to MPS, and actual commanded steps to Cobra 
FPGA are found to be quite important for both configuration (building active 
motor maps from statistics of real movement) and engineering (checking 
performance not degraded or hardware not damaged).
For these purpose, we have decided to store every operational data sets into 
the shared database, and started database design to cover not only 
operational point of view to keep records of commanded targets, 
but also configuration and/or engineering point of view. This database is 
fully integrated into the master database system for instrument operation 
on-site (at the summit; under spt_operational_database), and shared among 
the entire PFS ICS software modules. 
Acquired or calculated data will be (need to be?) put into the database 
in real time, so database schema optimization could be a key for processing 
time.

These data flows happen only during Cobra configuration sequence is running 
or validation of Cobra positions without moving, but not during exposure. 
Duration of Cobra configuration operation is dominated by each operation time 
of individual hardware, such as 1 to 5 seconds are assumed for MCS image 
acquisition, or a few seconds for Cobra movement. Their sequence is almost 
serial and timing for data exchange of such set of data points are once to 
several times at each proceeding point between hardware operations, so data 
flow will happen one or several times per several seconds (but not continuous 
nor in high frequency like 10Hz). 

For other ICS modules than FPS and modules under control, data sets to be used 
from this data exchange are:

- The final Cobra positions and their statistics need to be included in science 
  data FITS files (format and its way of data handling are TBD)
- Operational data are used for status viewer by instrument operators during 
  Cobras moving

Requirements in summary
======

Requirements on Cobra operational data handling in summary are:

- Make it possible to exchange several hundreds kB binary data without no side 
  effect to operation of other modules and with small overhead for data 
  acquisition and conversion
- Store every set of commanded or measured data into the operation database 
  in schema query-able by individual data point
- Provide a way to hand a set of final Cobra positions and statistics to 
  camera FITS file generation routines

Implementation ideas
******

Possible implementations
======

From items provided as the ICS infrastructure, possible implementations are as 
follows:

Serialization
  Serialize binary data as text and pass each as status over MHS, also insert 
  data into the operation database in parallel
Database
  Insert data into the operation database with an indexed key, and pass the 
  key as status over MHS

For a data set to FITS file generation, this part is better to be done by IIC 
rather than FPS, since we need to integrate such Cobra statistics with other 
information such as target information specified by observers or survey 
processing system. 

Details for Serialization
======

Status over MHS accepts arrays of numerical data, but these will be transferred 
as ',' separated array of strings, whose size will be quite larger than binary 
or its serialized string. Sending data in such normal format of MHS has a pros 
on data archive that these data in status messages will be archived into 
status archive database one by one as numerical value. 
Serialized data will be around 4/3 (BASE64) or 5/4 (BASE85; could require some 
replacement on encoded charactor to match with MHS requirement) of original 
binary data, its conversion time is negligible compared to other overheads 
like network communication. 
Therefore, for sending entire data over the MHS server, it is better to send 
by text serialized binary format on processing and network data flow points 
of view. 

Current implementation in the MHS server and client system for sharing status 
is to send statuses to each client in all or nothing basis, which means any 
client can select to reveice every statuses or not to receive anything, and 
a client library saves statuses received from the server into internal memory 
and arises events for status updates which a client code configured to listen. 
This makes the MHS server at production to copy every status lines quite much 
times and mostly not to be used. Even having 100 clients connected to the 
server, it makes just 100 times and several tens MB in total, but it will cost 
on network or the server itself. 

For saving Cobra reconfiguration data into the database, it shall be handled 
by some actor (could be FPS?) independently from its operation, but not require 
real time operation (not to be used for real time operation but for later use 
only). 

Details for Database
======

This is to use stored data in database directly inserted by each data 
generation actors, such as spot measurement statistics from MCS, calculated 
target positions from ETS/IIC, or converted target and current positions 
for Cobras by FPS, and to pass just type of data set and its identifier (like 
sequential ID taken from the database; such as one generated by a pair of 
a target exposure ID and Cobra movement sequence ID within one Cobra 
reconfiguration, depends on schema definition) over a command or a status over 
the MHS.
Sending only such identifier over the MHS will reduce data flow over the MHS 
server, but the database server will get some amount of data read from clients 
working for Cobra configuration operation. 

On a database point of view, a design of their schema need to be defined with 
much care, such as re-indexing on inserting data and efficient indexes for 
acquiring data, and also its performance optimization need to be validated. 
Total data volume into the database could be 100 configurations per night 
(including ones for calibration) or 1000 movements per night, which could be 
0.6M movements for whole instrument operation (assuming 600 nights). If we take 
one row per Cobra design, it will be a several times of 1.4G rows in total. 

Trades
******

Considering effects of distribution for status to the entire system, it is 
better to save data into the operation database directly and to handle only its 
pointer over the MHS. We may be possible to add mechanism for selection of 
messages (command or status) to be sent from the server to each client, but 
adding new layer at this moment could be harmful on its stability point of 
view and also could require modification on all actors and also careful 
inspection on defined sequences (to check which information are required by 
which actor). 

Therefore, we'd take a way to save data into the operation database and to 
exchange only its pointer. 

