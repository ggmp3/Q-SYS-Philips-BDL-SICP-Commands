# Q-SYS-Philips-BDL-SICP-Commands

- Written by Glen Gorton (glen.gorton@gmail.com)
- Built using a Philips 65BDL3511Q (SICP v2.06, Firmware V1.020T66064)

## NOTES:

- To establish communication via TCP/IP connect to display on port 5000. TCP/IP port 5000 is the default control port for all displays. Some displays have the option to change the communication port in the settings.
- Control commands can be sent from a host controller via the Ethernet (port 5000) connection. A new command should not be sent until the previous command is acknowledged.
- If a response is not received within 500 milliseconds a retry may be triggered. Have found this is not true -> a socket disconnect and reconnect is required.

- Command Packet format:
Byte 1: MsgSize  
Byte 2: Control / Monitor ID (set within the display = 1 to 255(0x01 to 0xFF))  
Byte 3: Group (Group ID range = 1 to 254(0x01 to 0xFE))
Byte 4 to Byte 39: Data[0] to Data[N]. Data parameters. This field can also be empty.  
Last Byte: Checksum (range = 0 to 255(0x00 to 0xFF))  


- See Command tables below for controls that were unable to be tested as they're not supported on the xxBDL3511Q models.
- Display will stop responding during power on/off. Polling will stop for 10 secs during this time, then restart.
- PollTimer is stopped prior to sending each command, and is then restarted after data is received.
- For NAV (Not Available) GetCommand responses, have included logic within the NAV Response to update the related Control.String to "n/a".

- 65DBL3511Q does not have Android sources 'Media Player, 'Browser' or 'PDF Player' -> unable to test Playlist/URL and USB Autoplay for these sources.
- 65BDL3511Q does have 'USB' source, which reports back as 'Media Player' when querying the display.
- Can confirm the Boot Source-> USB ->Playlist 1-7 and USB Autoplay does work provided the correct folder structure is created on the USB for photo/video:  
1. Create a folder named as “playlist1”, "playlist2", etc or "autoplay"
2. Put the sources(photo/video) within the folder. (There is no actual playlist file)

- Unable to test 'Send screenshot' function. Email server configuration not available on the 65BDL3511Q.

- Older models not running SICP v2.05 or later are unable to respond to getNUMBER_OF_INPUT_SOURCES which populates the 'Input Source' and 'Boot On Source' combo box and 'Number Of Input Sources' text box.
- If the 'Input Source' control pin is exposed, the script is still able to send discrete commands when a component such as a Selector is used to send input strings from the AllInputSources table (below).


## 24th June 2025 (v1.1)
- Created tables 'AllPhilipsInputSources' and 'AllPhilipsBootOnSources' that contains all input and boot sources based on the SICP v2.06 document.
- For displays that don't support the getNUMBER_OF_INPUT_SOURCES command they will respond with NAV or NACK. (ie. 43BDL4051T SICP v2.01 = NACK, 75BDL3151T SICP v2.04 = NAV)
- When NAV or NACK is returned the Controls will be udpdated:
- Controls["Input Source"].Choices = AllPhilipsInputSources
- Controls["Boot On Source"].Choices = AllPhilipsBootOnSources
- This allows the plugin to send the command for any input selected source, though the end user will need to know what sources are supported on the display.

- Modified the Initialize() function to select 0.5secs as the default Poll Interval (Controls["Poll Interval"].String = Controls["Poll Interval"].Choices[4])

- Moved getMODEL_NUMBER up the PollCommands table to be the 4th item polled to allow for some logic based on model#:
- Found 75BDL3151T and 43BDL4051T can fail when polled too frequently. Within the getMODEL_NUMBER response ("\xA1") added an if statement to adjust Controls["Poll Interval"].String based on model#.
- Found 75BDL3151T will not respond to getANDROID_VERSION. Within the getMODEL_NUMBER response ("\xA1"), Controls["Android Version"].String will update to "n/a"
- Found 75BDL3151T will not reliably respond to getVIDEO_PARAMETERS, getVOLUME, getBUILD_DATE. Decision made that 75BDL3151T will not function reliably with this plugin.

- Within PollTimer.EventHandler, added logic to remove certain items from Polling for the 75BDL3151T. CurrentPollCommand will be increased by 1 (or reset to 1) to skip over the unsupported commands (getANDROID_VERSION,  getSTORAGELOCK)

## 1st July 2025
- Found the 43BDL4051T does not respond with an 'Acknowledge' for Power On, preventing the polling being paused, resulting in a Compromised status being set.
- Within the Controls["Power"].EventHandler, added model# based logic for the 43BDL4051T to pause all polling and restart after 60 seconds.


Philips Input Sources Table
AllInputSources = {
  [1] = "VIDEO",
  [2] = "S-VIDEO",
  [3] = "COMPONENT",
  [4] = "CVI 2", --(not applicable)
  [5] = "VGA",
  [6] = "HDMI 2",
  [7] = "DisplayPort 2",
  [8] = "USB 2",
  [9] = "Card DVI-D",
  [10] = "Display Port 1",
  [11] = "Card OPS",
  [12] = "USB 1",
  [13] = "HDMI",
  [14] = "DVI-D",
  [15] = "HDMI 3",
  [16] = "BROWSER",
  [17] = "SMART CMS",
  [18] = "DMS (Digital Media Server)",
  [19] = "INTERNAL STORAGE",
  [20] = "Reserved 1",
  [21] = "Reserved 2",
  [22] = "Media Player",
  [23] = "PDF Player",
  [24] = "Custom",
  [25] = "HDMI 4",
  [26] = "VGA 2",
  [27] = "VGA 3",
  [28] = "IWB",
  [29] = "CMND & Play Web",
  [30] = "Home/Launcher",
  [31] = "USB Type-C 1",
  [32] = "Kiosk",
  [33] = "Smart Info",
  [34] = "Tuner",
  [35] = "Google Cast",
  [36] = "Interact",
  [37] = "USB Type-C 2",
}

GetCommands = {  
  getNUMBER_OF_INPUT_SOURCES = {0xAB}, -- SICP v2.05 onwards  
  getINPUT_SOURCE = {0xAD},
  getBOOT_SOURCE = {0xBA}, -- SICP v2.05 onwards
  getMODEL_NUMBER = {0xA1, 0x00},
  getVIDEO_PARAMETERS = {0x33},
  getPOWER_STATE = {0x19},  
  getSICP_VERSION = {0xA2, 0x00},  
  getPLATFORM_LABEL = {0xA2, 0x01},  
  getPLATFORM_VERSION = {0xA2, 0x02}, 
  getFIRMWARE_VERSION = {0xA1, 0x01},  
  getBUILD_DATE = {0xA1, 0x02},  
  getANDROID_VERSION = {0xA1, 0x03}, -- NAV - Not Available on xxBDL3511Q -- NAV - Not Available: Command is valid but not supported in the current SICP implementation.  
  getHDMI_SWITCH_VERSION = {0xA1, 0x04}, -- SICP v2.09 onwards -- NAV - Not Available: Command is valid but not supported in the current SICP implementation.  
  getLAN_FW_VERSION = {0xA1, 0x05}, -- SICP v2.09 onwards -- NAV - Not Available: Command is valid but not supported in the current SICP implementation.  
  getHDMI_SWITCH2_VERSION = {0xA1, 0x06}, -- SICP v2.10 onwards -- NAV - Not Available: Command is valid but not supported in the current SICP implementation.  
  getOPERATING_HOURS = {0x0F, 0x02},  
  getTEMPERATURE_SENSOR = {0x2F},  
  getSERIAL_NUMBER = {0x15},  
  getVIDEO_SIGNAL_PRESENT = {0x59}, -- SICP v2.03 onwards  
  getPOWER_STATE = {0x19},  
  getCOLD_START_STATE = {0xA4},  
  getPOWER_SAVE_MODE = {0xD3},  
  getSMART_POWER_MODE = {0xDE},  
  getADVANCED_POWER_MANAGEMENT = {0xD1}, -- Supported on Himalaya & Eagle 1.3 Platforms. NACK - Not Acknowledged ON xxBDL3511Q  
  getECO_MODE = {0x63}, -- SICP v2.00 onwards. NACK - Not Acknowledged: Checksum/Format error on xxBDL3511Q  
  getBACKLIGHT = {0x71}, -- SICP v2.03 onwards  
  getOPS_POWER_SETTING = {0x6E}, -- SICP v2.08 onwards -- NACK - Not Acknowledged: Checksum/Format error. Unable to test on xxBDL3511Q running SICP v2.06  
  getSIGNAL_AUTO_DETECTION = {0xAF},  
  getVOLUME = {0x45},  
  getMUTE = {0x46}, -- SICP v2.05 onwards. Mutes both the internal speakers and the audio output.  
  getSPEAKERS = {0x8F}, -- SICP v2.07 onwards. -- NACK - Not Acknowledged: Checksum/Format error. Unable to test on xxBDL3511Q running SICP v2.06  
  getREMOTELOCK = {0x1D},  
  getKEYPADLOCK = {0x1B},  
  getPICTURE_FORMAT = {0x3B},  
  getFREEZEIMAGE = {0x76}, -- SICP v2.06 onwards.  
  getSTORAGELOCK = {0xF2}, -- NACK - Not Acknowledged: Checksum/Format error on xxBDL3511Q  
}  

SetCommands = {  
  setPOWER_STATE = {0x18},  
  setCOLD_START_STATE = {0xA3},  
  setPOWER_SAVE_MODE = {0xD2},  
  setSMART_POWER_MODE = {0XDD},  
  setADVANCED_POWER_MANAGEMENT = {0xD0}, -- Supported on Himalaya & Eagle 1.3 Platforms. NACK - Not Acknowledged ON xxBDL3511Q  
  setECO_MODE = {0x64}, -- SICP v2.00 onwards. NACK - Not Acknowledged: Checksum/Format error on xxBDL3511Q  
  setMONITOR_RESTART = {0x57}, -- SICP v2.02 onwards. This command is used to restart/reboot the display.  
  setBACKLIGHT = {0x72}, -- SICP v2.03 onwards  
  setOPS_POWER_SETTING = {0x6F}, -- SICP v2.08 onwards -- NACK - Not Acknowledged: Checksum/Format error. Unable to test on xxBDL3511Q running SICP v2.06  
  setINPUT_SOURCE = {0xAC},  
  setBOOT_SOURCE = {0xBB}, -- SICP v2.05 onwards  
  setSIGNAL_AUTO_DETECTION = {0xAE},  
  setVOLUME = {0x44}, -- Speaker Volume and Audio Out Volume values are inserted at the Volume Eventhandler.  
  setVOLUME_STEP = {0x41}, -- Speaker Volume and Audio Out Volume values are inserted at the Volume Step Eventhandler.  
  setMUTE = {0x47}, -- SICP v2.05 onwards. Mutes both the internal speakers and the audio output.  
  setSPEAKERS = {0x8E}, -- SICP v2.07 onwards. -- NACK - Not Acknowledged: Checksum/Format error. Unable to test on xxBDL3511Q running SICP v2.06  
  setREMOTELOCK = {0x1C},  
  setKEYPADLOCK = {0x1A},  
  setVIDEO_PARAMETERS = {0x32},  
  setPICTURE_FORMAT = {0x3A},  
  setFREEZEIMAGE = {0x77}, -- SICP v2.06 onwards.  
  setSTORAGELOCK = {0xF1}, -- NACK - Not Acknowledged: Checksum/Format error on xxBDL3511Q  
  setSEND_SCREENSHOT = {0x58}, -- SICP v2.02 onwards. This command is used to Take a screenshot of current source and send it via Email. NACK - Not Acknowledged: Checksum/Format error on xxBDL3511Q  
}  

PollCommands = {  
  {0xAB}, --getNUMBER_OF_INPUT_SOURCES -- SICP v2.05 onwards  
  {0xAD}, --getINPUT_SOURCE  
  {0xBA}, --getBOOT_SOURCE -- SICP v2.05 onwards
  {0xA1, 0x00}, --getMODEL_NUMBER  
  {0x33}, --getVIDEO_PARAMETERS  
  {0x19}, --getPOWER_STATE  
  {0xA2, 0x00}, --getSICP_VERSION  
  {0xA2, 0x01}, --getPLATFORM_LABEL  
  {0xA2, 0x02}, --getPLATFORM_VERSION
  {0xA1, 0x01}, --getFIRMWARE_VERSION  
  {0xA1, 0x02}, --getBUILD_DATE  
  {0xA1, 0x03}, --getANDROID_VERSION -- NAV - Not Available on xxBDL3511Q  
  {0xA1, 0x04}, --getHDMI_SWITCH_VERSION -- SICP v2.09 onwards -- NAV - Not Available on xxBDL3511Q  
  {0xA1, 0x05}, --getLAN_FW_VERSION -- SICP v2.09 onwards -- NAV - Not Available on xxBDL3511Q  
  {0xA1, 0x06}, --getHDMI_SWITCH2_VERSION -- SICP v2.10 onwards -- NAV - Not Available on xxBDL3511Q  
  {0x0F, 0x02}, --getOPERATING_HOURS  
  {0x2F}, --getTEMPERATURE_SENSOR  
  {0x15}, --getSERIAL_NUMBER  
  {0x59}, --getVIDEO_SIGNAL_PRESENT -- SICP v2.03 onwards  
  {0xA4}, --getCOLD_START_STATE  
  {0xD3}, --getPOWER_SAVE_MODE  
  {0xDE}, --getSMART_POWER_MODE  
  {0xD1}, --getADVANCED_POWER_MANAGEMENT -- Supported on Himalaya & Eagle 1.3 Platforms. NACK - Not Acknowledged ON xxBDL3511Q  
  {0x63}, --getECO_MODE -- SICP v2.00 onwards. NACK - Not Acknowledged: Checksum/Format error on xxBDL3511Q  
  {0x71}, --getBACKLIGHT -- SICP v2.03 onwards  
  {0x6E}, --getOPS_POWER_SETTING -- SICP v2.08 onwards -- NACK - Not Acknowledged: Checksum/Format error. Unable to test on xxBDL3511Q running SICP v2.06  
  {0xAF}, --getSIGNAL_AUTO_DETECTION  
  {0x45}, --getVOLUME  
  {0x46}, --getMUTE -- SICP v2.05 onwards. Mutes both the internal speakers and the audio output.  
  {0x8F}, --getSPEAKERS -- SICP v2.07 onwards. -- NACK - Not Acknowledged: Checksum/Format error. Unable to test on xxBDL3511Q running SICP v2.06  
  {0x1D}, --getREMOTELOCK  
  {0x1B}, --getKEYPADLOCK  
  {0x3B}, --getPICTURE_FORMAT  
  {0x76}, --getFREEZEIMAGE -- SICP v2.06 onwards.  
  {0xF2}, --getSTORAGELOCK -- NACK - Not Acknowledged: Checksum/Format error on xxBDL3511Q  
} 
