# CBT662
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 662 is from Dave Cartwright and contains a system to      *   FILE 662
//*           provide an automated bridge between CA-1 and FLEX-ES  *   FILE 662
//*           Faketape.  A detailed description of this system      *   FILE 662
//*           follows below.                                        *   FILE 662
//*                                                                 *   FILE 662
//*     Comments and ideas for improvement are welcomed,            *   FILE 662
//*     email to:                                                   *   FILE 662
//*                                                                 *   FILE 662
//*     davecartwright@uk.agcocorp.com                              *   FILE 662
//*                                                                 *   FILE 662
//*  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  *   FILE 662
//*                                                                 *   FILE 662
//*     This file contains code used to provide an automated        *   FILE 662
//*     bridge between the CA-1 tape management system and the      *   FILE 662
//*     FakeTape(tm) facility of FLEX-ES.  This contribution        *   FILE 662
//*     will require a lot of tailoring to your environment.        *   FILE 662
//*                                                                 *   FILE 662
//*     It is based upon the TSSO Automated Operator Facility       *   FILE 662
//*     (AOF), for which see File 404 of the CBT tape.  It uses     *   FILE 662
//*     a network connection between the MVS system and Unixware    *   FILE 662
//*     (or Linux) to issue remote commands which create the        *   FILE 662
//*     FakeTape(tm) file and MOUNT it on the desired unit          *   FILE 662
//*     address. This may require a network bridge to route         *   FILE 662
//*     traffic between the networks and may need TCP/IP routing    *   FILE 662
//*     changes in MVS and in Unixware (or Linux).  The members     *   FILE 662
//*     are;                                                        *   FILE 662
//*                                                                 *   FILE 662
//*     $DOC     -  You are reading it.                             *   FILE 662
//*                                                                 *   FILE 662
//*     FAKEAOF  -  TSSO AOF TABENTRY statements for CA-1           *   FILE 662
//*                 messages.  Include these with a COPY            *   FILE 662
//*                 FAKEAOF statement in AOF source.                *   FILE 662
//*                                                                 *   FILE 662
//*     FSF      -  Assembler source for a program which reads      *   FILE 662
//*                 the CA-1 TMC to find the first available        *   FILE 662
//*                 SCRATCH tape. The characteristics of the TMC    *   FILE 662
//*                 have to be specified as Assembler variables     *   FILE 662
//*                 at the start of the program. You will need      *   FILE 662
//*                 macros from File 172 of the CBT tape and        *   FILE 662
//*                 from CA-1 to be able to assemble this           *   FILE 662
//*                 program. This program invokes the program       *   FILE 662
//*                 "SETVAR" from File 270 of the CBT tape to       *   FILE 662
//*                 return the volume serial in the TSO variable    *   FILE 662
//*                 "FAKETAPE".                                     *   FILE 662
//*                                                                 *   FILE 662
//*     FTAPE     - A clist which is invoked under TSSO to          *   FILE 662
//*                 process requests for SCRATCH tapes. It tests    *   FILE 662
//*                 whether the tape unit is on a string of         *   FILE 662
//*                 tapes defined for FakeTape(tm) use and if so    *   FILE 662
//*                 it invokes FSF to find a SCRATCH tape. Then     *   FILE 662
//*                 it issues a REXEC to the Unixware (or Linux)    *   FILE 662
//*                 Userid "flexes" with the appropriate            *   FILE 662
//*                 password to create a file with a VOL1 label     *   FILE 662
//*                 for that tape.  That VOL1 is then converted     *   FILE 662
//*                 into a FakeTape(tm) file in the appropriate     *   FILE 662
//*                 directory, which is then MOUNTed by FLEX-ES     *   FILE 662
//*                 on the right unit address. MVS opens the        *   FILE 662
//*                 file, checks the volume serial in the label     *   FILE 662
//*                 VOL1, CA-1 checks that it is a SCRATCH tape,    *   FILE 662
//*                 updates the TMC and the tape file is then       *   FILE 662
//*                 written. Code is included to cater for          *   FILE 662
//*                 multiple simultaneous SCRATCH mounts with       *   FILE 662
//*                 some crude placemat allocation. These files     *   FILE 662
//*                 should be regularly monitored lest they         *   FILE 662
//*                 proliferate.  Note that Unixware (and Linux)    *   FILE 662
//*                 commands are case sensitive.                    *   FILE 662
//*                                                                 *   FILE 662
//*     STAPE     - A clist which is invoked under TSSO to          *   FILE 662
//*                 process requests for specific tapes. It         *   FILE 662
//*                 tests whether the tape unit is one defined      *   FILE 662
//*                 for FakeTape(tm) use and if so issues a         *   FILE 662
//*                 REXEC to the Unixware (or Linux) Userid         *   FILE 662
//*                 "flexes" with the appropriate password to       *   FILE 662
//*                 MOUNT the appropriate file by FLEX-ES on the    *   FILE 662
//*                 right unit address. MVS opens the file,         *   FILE 662
//*                 checks the volume serial in the label and       *   FILE 662
//*                 processes the "tape".  Note that Unixware       *   FILE 662
//*                 (and Linux) commands are case sensitive.        *   FILE 662
//*                                                                 *   FILE 662
//*     FDONE     - A clist which is invoked under TSSO to          *   FILE 662
//*                 process requests to demount tapes. It           *   FILE 662
//*                 attempts to delete the placemat file for        *   FILE 662
//*                 that volume. It does not care whether the       *   FILE 662
//*                 placemat exists or not - it will cease to       *   FILE 662
//*                 do so anyway.                                   *   FILE 662
//*                                                                 *   FILE 662
//*     $ZDOC     - Documentation on archiving experiments          *   FILE 662
//*     VTAPE1    - See $ZDOC                                       *   FILE 662
//*     VTAPE2    - See $ZDOC                                       *   FILE 662
//*     ZTAPE     - See $ZDOC                                       *   FILE 662
//*     ZDONE     - See $ZDOC                                       *   FILE 662
//*     ZSTAPE    - See $ZDOC                                       *   FILE 662
//*                                                                 *   FILE 662
//*     AGCO UK Ltd. replaced an IBM Multiprise 2003 mainframe      *   FILE 662
//*     running OS390 R2.10 in February 2004 with an IBM            *   FILE 662
//*     X-series server running FLEX-ES under SCO Unixware. The     *   FILE 662
//*     same OS390 R2.10 system was copied across. We run a         *   FILE 662
//*     mixed workload with a few TSO Users, about 60 IMS Users     *   FILE 662
//*     and overnight batch. We have a worldwide network which      *   FILE 662
//*     was SNA but is now mostly IP.  We have successfully         *   FILE 662
//*     tested Enterprise Extender over the FLEX-ES emulated OSA    *   FILE 662
//*     card.  We ordered a three channel adaptor card supplied     *   FILE 662
//*     by Fundamental Software Inc., the vendors of FLEX-ES to     *   FILE 662
//*     drive a 3745, local printers and tapes.                     *   FILE 662
//*                                                                 *   FILE 662
//*     We installed four daisy chained SCSI tape decks as well     *   FILE 662
//*     as having eight IBM 3490-E drives connected via one         *   FILE 662
//*     channel of the PCA.  As may be expected tape performance    *   FILE 662
//*     is poor, but it has always been our intention to replace    *   FILE 662
//*     tapes with FakeTape(tm) anyway. To that end we defined      *   FILE 662
//*     in the IODF a string of 3480 tape drives for use with       *   FILE 662
//*     this facility.  We also defined an additional EDT with      *   FILE 662
//*     the MIS standard name CART pointing to these drives.        *   FILE 662
//*     Thus we can switch FakeTape(tm) on and off by activating    *   FILE 662
//*     the appropriate EDT, either at IPL or by a dynamic          *   FILE 662
//*     software-only change using HCD. Because they have           *   FILE 662
//*     different device types in their catalog entries there is    *   FILE 662
//*     no confusion of fake or real tapes. To be sure we use a     *   FILE 662
//*     separate range of tape numbers in the TMC for               *   FILE 662
//*     FakeTape(tm).                                               *   FILE 662
//*                                                                 *   FILE 662
```
