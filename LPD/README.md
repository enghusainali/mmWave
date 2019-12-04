# mmWave-LPD (Low range People Detect)-ES2.0 (Coming Soon...)
# Notes: mmWave Library supports: python Version >= 3.6

Current PI's OS is supports python 3.7.0

https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up/3

This repository contains the Batman Kit- 201(ISK) Sensing mmWave Sensor SDK Device Version:ES2.0 .  The sample code below consists of instruction for using the mmWave lib. This mmWave-LPD Python Program will work with Long range  based Batman BM201 mmWave Kit solution (BM201-LPD). This Python Program works with a Raspberry Pi 4 and/or NVIDIA Jetson Nano computer with Batman BM301-POC or BM301-FDS Kit attached; and that  the BM301 Kit is an easy-to-use mmWave sensor evaluation kit with miniaturized short-range antenna, and with wide horizontal and vertical Field of View (FoV), that connects directly to a Raspberry Pi or NVIDIA Jetson Nano computer via Kit's HAT Board, for detecting multiple objects in a 3-Dimentional Area with ID tag, posX, posY, posZ, Vx, Vy, Vz, accX, accY, accZ parameters and Point Clouds with elevation, azimuth, doppler, range, and snr parameters.

Hardware Sensor: 

    Batman BM201-LPD provid two types of data:

        BM201-LPD provids raw data as:

        Point Cloud Spherical(V6): range,azimuth,elevation,doppler
        Target Object (V7): tid,posX,posY,velX,velY,accX,accY,posZ,velZ,accZ
        Target Index (V8): tid and status
        Point Cloud Side Info (V9): snr,noise
    
        
# Physical Setup
        Setup Requirements:
        Elevate EVM:2.0~2.5m high
        Down tilt:~2-3 degree
![MainMenu 1](https://github.com/bigheadG/mmWave/blob/master/physical_setup_lpd.png)

        
# Installing

Library install for Python

    $sudo pip3 install mmWave

Library update:

    $sudo pip3 install mmWave -U

# Examples:
    Raw data:
        lpd_v09_raw_ex0.py is a example output V6(Point Cloud Spherical),V7(Target Object List),V8(Target Index), V9(Point Cloud Side Info) data
        
    key/value:
        

# Data Structure(Raw Data):
## Lib import(Raw Data):

from mmWave import lpdISK
lpd = lpdISK.LpdISK(port)
   
    V6: Point Cloud Spherical
    Each Point Cloud list consists of an array of points, Each point data structure is defined as following
   
    point Struct:
        range:    float   #Range in meters
        azimuth:  float   #Azimuth in radians
        elevation: float  #Elevation in radians
        doppler:  float   #Doppler in m/s
    
        V6 =: [(range,azimuth,elevation,doppler),......]
        
    V7: Target Object
    Each Target List consists of an array of targets. Each target data structure defind as following:
    
    target Struct:
        tid: Int        #Track ID
        posX: float     #Target position in X, m
        posY: float     #Target position in Y, m
        velX: float     #Target velocity in X, m/s
        velY: float     #Target velocity in Y, m/s
        accX: float     #Target velocity in X, m/s
        accY: float     #Target velocity in Y, m/s
        posZ: float     #Target position in Z, m
        velZ: float     #Target velocity in Z, m/s
        accZ: float     #Target velocity in Z, m/s
        
        V7 =: [(tid,posX,posY,velX,velY,accX,accY,posZ,velZ,accZ),....]
        
    V8: Target Index
    Each Target List consists of an array of target IDs, A targetID at index i is the target to which point i of the previous frame's point cloud was associated. Valid IDs range from 0-249
        
    TargetIndex Struct:
        tragetID: Int #Track ID
        [targetID0,targetID1,.....targetIDn]
        
        Other Target ID values:
        253:Point not associated, SNR to weak
        254:Point not associated, located outside boundary of interest
        255:Point not associated, considered as noise
   
        V8 =: [id1,id2....]
    
    V9:Point Cloud Side Info
        v9 structure: [(snr,noise'),....]    
        
    Function call: 
        (dck,v6,v7,v8,v9) = lpd.tlvRead(False)
        dck : True  : data avaliable
              False : data invalid
        v6: point cloud of array
        v7: target object of array
        v8: target id of array
        v9: point cloud side info array

        return dck,v6,v7,v8,v9
		
   	 -----------------
	 |  obj o  |+z  board
	 |       \ |
	 |      el\|
	 | +x<-----o
	 |   az     \
	 ----------- \ --
	 	      \+y
    
    Based on IWR6843 (r,az,el) -> (x,y,z)
    el: elevation  <Theta bottom -> Obj  
    az: azimuth    <Theta Obj ->Y Axis 
    
    z = r * sin(el)
    x = r * cos(el) * sin(az)
    y = r * cos(e1) * cos(az)
 
 # Data Structure(Key/Value):

 
    Detected Object Data Format: <frameNum,numObjs,[op]>
    frameNum: Frame Number
    numObjs: number of Detected Object
    [op]: an Array of Detected Objects Point , the point includes position and object moving speed data
    op: objPoint class
    <frameNum,numObjs,op=[(tid0,x0,y0,z0,vx0,vy0,vz0,accx0,accy0,accz0,state0),(tid1,x1,y1,z1,vx1,vy1,vz1,accx1,accy1,accz1,state1),(tid2,x2,y2,z2,vx2,vy2,vz2,accx2,accy2,accz2,state2)...]>
    
    @dataclass
    class objPoint:
        tid: int 
        x: float
        y: float
        z: float = 0.0
        vx: float = 0.0
        vy: float = 0.0
        vz : float = 0.0
        accX : float = 0.0
        accY : float = 0.0
        accZ : float = 0.0
        state : int = 0
 
    notes:
        state: 1:stand 2:seat 3:lie down 5:falling
        
    @dataclass
    class objSets:
        frameNum: int
        numObjs: int
        op: [objPoint]


# import lib
    raw data:
    
    from mmWave import lpdISK
    

  ### raspberry pi 3/pi 4 use ttyS0
    port = serial.Serial("/dev/ttyS0",baudrate = 921600, timeout = 0.5)
    
  ### Jetson Nano use ttyTHS1
	
	port = serial.Serial("/dev/ttyTHS1",baudrate = 921600, timeout = 0.5)
	and please modify: 
	
	#import RPi.GPIO as GPIO
	import Jetson.GPIO as GPIO

## define
    raw data:
    
    lpd = lpdISK.LpdISK(port)
    
    key/value:
    
     

## get LPD-ISK Sensor Data

    raw data:
    
        (dck,v6,v7,v8,v9)  = lpd.tlvRead(False)
        if dck:
            print(v6) #you will see v6 data
        
    
    
## Reference:
	
    

