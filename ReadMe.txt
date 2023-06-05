To run this program you need the .ap15_1 file, which sadly can't be uploaded to github.

The programming language is SCL, short for Structured Control Language, built on PASCAL
and used for advanced programming.

This runs simultaneously with another program called NX Siemens

However, we can show the code here.
If you need more information about this program please contact the owner,
or copy the following link to see the simulation: 
https://www.youtube.com/watch?v=LVDSWr5Uuf4

//This case is for manual operation of the production line
CASE #ManualProduction OF
    0:
        //Everything is set to false/default position
        "Conveyor" := False;
        "WorkpieceBlack" := False;
        "WorkpieceRed" := False;
        "WorkpieceSilver" := False;
        "PistonLinearHeadExtend" := True;
        "PistonLinearHeadRetract" := False;
        "PistonRotativeHead1Extend" := False;
        "PistonRotativeHead1Retract" := True;
        "PistonRotativeHead2Extend" := False;
        "PistonRotativeHead2Retract" := True;
        "BlackInPlay" := False;
        "RedInPlay" := False;
        "SilverInPlay" := False;
        "TimerStartStopper" := False;
        "TimerFinishStopper" := False;
        
        //To optionally unload slide1 if full
        IF "ClearSlide1Button" AND "Slide1Full" THEN
            "SinkSlide1" := True;
            "Slide1Full" := False;
        END_IF;
        
        //To optionally unload slide2 if full
        IF "ClearSlide2Button" AND "Slide2Full" THEN
            "SinkSlide2" := True;
            "Slide2Full" := False;
        END_IF;
            
        //Insert a workpiece, first it asks if both orders are 6 units or less and if...
        //...is not in automatic mode
        IF ("CounterWorkpieceBlackSlide1" + "CounterWorkpieceRedSlide1"
            + "CounterWorkpieceSilverSlide1") <= 6 AND ("CounterWorkpieceBlackSlide2" +
            "CounterWorkpieceRedSlide2" + "CounterWorkpieceSilverSlide2") <= 6 AND
            #AutomaticProduction = 0 THEN
            
            //To insert a black workpiece
            IF "BlackWorkpieceButton" AND "LightSensorWorkpieceSlide" = False THEN
                #ManualProduction := 1;
            END_IF;
            
            //To insert a red workpiece
            IF "RedWorkpieceButton" AND "LightSensorWorkpieceSlide" = False THEN
                #ManualProduction := 2;
            END_IF;
            
            //To insert a silver workpiece
            IF "SilverWorkpieceButton" AND "LightSensorWorkpieceSlide" = False THEN
                #ManualProduction := 3;
            END_IF;
        END_IF;
        
        //To see which message to show depeding if orders are full
        IF ("CounterWorkpieceBlackSlide1" + "CounterWorkpieceRedSlide1"
            + "CounterWorkpieceSilverSlide1") > 6 OR ("CounterWorkpieceBlackSlide2" +
            "CounterWorkpieceRedSlide2" + "CounterWorkpieceSilverSlide2") > 6 THEN
            "MessageDisplay" := 17; //Order must be 6 units OR less
        ELSIF "Slide1Full" = False AND "Slide2Full" = False THEN
            "MessageDisplay" := 0; //"Waiting for instructions..."
        ELSIF "Slide1Full" = True AND "Slide2Full" = False THEN
            "MessageDisplay" := 13; //"Waiting for instructions. Order #1 is ready to collect"
        ELSIF "Slide1Full" = False AND "Slide2Full" = True THEN
            "MessageDisplay" := 14; //"Waiting for instructions. Order #2 is ready to collect"
        ELSIF "Slide1Full" = True AND "Slide2Full" = True THEN
            "MessageDisplay" := 15; //"Waiting for instructions. Orders #1 & 2 are ready to collect"
        END_IF;
        
        //If automatic or control process is active, then this case will be sent to  #10...
        //... to prevent conflicts
        IF #AutomaticProduction >= 1 OR #ControlProduction >= 1 THEN
            #ManualProduction := 10;
        END_IF;
        
    1: //This case will only execute if black workpiece button is pressed
        "WorkpieceBlack" := True;
        IF "LightSensorWorkpiece" THEN
            #ManualProduction := 4;
            "WorkpieceBlack" := False;
            "Conveyor" := True; 
            "MessageDisplay" := 16; //"Workpiece detected"
        END_IF;
        
    2: //This case will only execute if red workpiece button is pressed
        "WorkpieceRed" := True;
        IF "LightSensorWorkpiece" THEN
            #ManualProduction := 4;
            "WorkpieceRed" := False;
            "Conveyor" := True; 
            "MessageDisplay" := 16; //"Workpiece detected"
        END_IF;
        
    3:  //This case will only execute if silver workpiece button is pressed
        "WorkpieceSilver" := True;
        IF "LightSensorWorkpiece" THEN
            #ManualProduction := 4;
            "WorkpieceSilver" := False;
            "Conveyor" := True; 
            "MessageDisplay" := 16; //"Workpiece detected"
        END_IF;
        
    4:  //This case is to retract linear piston, and activate the...
        //...respective rotative piston
        "SinkSlide1" := False; //To deactivate sink, using this line in case 0 causes bugs
        "SinkSlide2" := False; //To deactivate sink, using this line in case 0 causes bugs
        "SinkSlide3" := False; //To deactivate sink, using this line in case 0 causes bugs
        
        //This IF is to turn off the conveyor until the stopper is retracted
        IF ("LightSensorWorkpieceBlack1" OR "LightSensorWorkpieceRed2" OR "LightSensorWorkpieceSilver3")
            AND "TimerFinishStopper" = False THEN
            "Conveyor" := False;
        ELSE
            "Conveyor" := True;
        END_IF;
            
        
        //When Black workpiece is detected 
        IF "LightSensorWorkpieceBlack1" THEN
            "BlackInPlay" := True;
            "TimerStartStopper" := True;
            IF "TimerFinishStopper" THEN //This is to delay the stopper a bit before opening
                "PistonLinearHeadExtend" := False;
                "PistonLinearHeadRetract" := True;
                IF "CounterWorkpieceBlackSlide1" > 0 THEN
                    "PistonRotativeHead1Extend" := True;
                    "PistonRotativeHead1Retract" := False;
                    "MessageDisplay" := 2;//"Black product going into Order #1"
                ELSIF "CounterWorkpieceBlackSlide2" > 0 THEN
                    "PistonRotativeHead2Extend" := True;
                    "PistonRotativeHead2Retract" := False;
                    "MessageDisplay" := 3;//"Black product going into Order #2"
                ELSE
                    "MessageDisplay" := 4;//"Black product not ordered, going into Station #3"
                END_IF;
            END_IF;
        END_IF;
        
        //When Red workpiece is detected 
        IF "LightSensorWorkpieceRed2" THEN
            "RedInPlay" := True;
            "TimerStartStopper" := True;
            IF "TimerFinishStopper" THEN //This is to delay the stopper a bit before opening
                "PistonLinearHeadExtend" := False;
                "PistonLinearHeadRetract" := True;
                IF "CounterWorkpieceRedSlide1" > 0 THEN
                    "PistonRotativeHead1Extend" := True;
                    "PistonRotativeHead1Retract" := False;
                    "MessageDisplay" := 5;//"Red product going into Order #1"
                ELSIF "CounterWorkpieceRedSlide2" > 0 THEN
                    "PistonRotativeHead2Extend" := True;
                    "PistonRotativeHead2Retract" := False;
                    "MessageDisplay" := 6;//"Red product going into Order #2"
                ELSE
                    "MessageDisplay" := 7;//"Red product not ordered, going into Station #3"
                END_IF;
            END_IF;
        END_IF;
        
        //When Silver workpiece is detected 
        IF "LightSensorWorkpieceSilver3" THEN
            "SilverInPlay" := True;
            "TimerStartStopper" := True;
            IF "TimerFinishStopper" THEN //This is to delay the stopper a bit before opening
                "PistonLinearHeadExtend" := False;
                "PistonLinearHeadRetract" := True;
                IF "CounterWorkpieceSilverSlide1" > 0 THEN
                    "PistonRotativeHead1Extend" := True;
                    "PistonRotativeHead1Retract" := False;
                    "MessageDisplay" := 8;//"Silver product going into Order #1"
                ELSIF "CounterWorkpieceSilverSlide2" > 0 THEN
                    "PistonRotativeHead2Extend" := True;
                    "PistonRotativeHead2Retract" := False;
                    "MessageDisplay" := 9;//"Silver product going into Order #2"
                ELSE
                    "MessageDisplay" := 10;//"Silver product not ordered, going into Station #3"
                END_IF;
            END_IF;
        END_IF;
        
        //When any workpiece goes through any slide
        IF "LightSensorWorkpieceSlide" THEN
            //If piston for slide1 is active
            IF "LimitSwitchPistonRotativeHead1Extended" THEN
                
                //Substract the corresponding colour counter in slide1
                IF "BlackInPlay" THEN     //Checks if black workpiece was placed
                    "CounterWorkpieceBlackSlide1" := "CounterWorkpieceBlackSlide1" - 1;
                ELSIF "RedInPlay" THEN    //Checks if red workpiece was placed
                    "CounterWorkpieceRedSlide1" := "CounterWorkpieceRedSlide1" - 1;
                ELSIF "SilverInPlay" THEN //Checks if silver workpiece was placed
                    "CounterWorkpieceSilverSlide1" := "CounterWorkpieceSilverSlide1" - 1;
                END_IF;
                
                //If piston for slide2 is active
            ELSIF "LimitSwitchPistonRotativeHead2Extended" THEN
                
                //Substract the corresponding colour counter in slide2
                IF "BlackInPlay" THEN
                    "CounterWorkpieceBlackSlide2" := "CounterWorkpieceBlackSlide2" - 1;
                ELSIF "RedInPlay" THEN
                    "CounterWorkpieceRedSlide2" := "CounterWorkpieceRedSlide2" - 1;
                ELSIF "SilverInPlay" THEN
                    "CounterWorkpieceSilverSlide2" := "CounterWorkpieceSilverSlide2" - 1;
                END_IF;
            END_IF;
            
            #ManualProduction := 5;
        END_IF;
    5: //This will retract both rotative pistons to its original position...
       //... and check if any slide is completed/full
       
        //Checking if Slide3 is full
        IF "LimitSwitchPistonRotativeHead1Extended" = False AND
            "LimitSwitchPistonRotativeHead2Extended" = False AND
            "TimerSlideFullFinish" = True THEN
            "Slide3Full" := True;
            #ManualProduction := 6;
        ELSIF "LimitSwitchPistonRotativeHead1Extended" = False AND
            "LimitSwitchPistonRotativeHead2Extended" = False AND
            "LightSensorWorkpieceSlide" = False THEN
            #ManualProduction := 6;
        END_IF;
        
        //Checking if Slide1 was active
        IF "LimitSwitchPistonRotativeHead1Extended" THEN
            //Checking if Slide1 order is completed/full
            IF ("CounterWorkpieceBlackSlide1" = 0 AND
                "CounterWorkpieceRedSlide1" = 0 AND
                "CounterWorkpieceSilverSlide1" = 0) OR "TimerSlideFullFinish" THEN
                "Slide1Full" := True;
            END_IF;
            
            //Resetting piston for Slide1
            "PistonRotativeHead1Extend" := False;
            "PistonRotativeHead1Retract" := True;
            #ManualProduction := 6;
        END_IF;
        
        //Checking if slide2 was active
        IF "LimitSwitchPistonRotativeHead2Extended" THEN
            //Checking if Slide2 order is completed/full
            IF ("CounterWorkpieceBlackSlide2" = 0 AND
                "CounterWorkpieceRedSlide2" = 0 AND
                "CounterWorkpieceSilverSlide2" = 0) OR "TimerSlideFullFinish" THEN
                "Slide2Full" := True;
            END_IF;
            
            //Resetting piston for Slide2
            "PistonRotativeHead2Extend" := False;
            "PistonRotativeHead2Retract" := True;
            #ManualProduction := 6;
        END_IF;
        
        //To reset linear piston
        "PistonLinearHeadExtend" := True;
        "PistonLinearHeadRetract" := False;
        
    6:  //This case is to verify  if slides 1 and 2 are simultaneously full,...
        //or if slide3 is full. If any condition is true it will be sent to case 7 or 8
        
        //Checking if slide3 is full
        IF "Slide3Full" THEN
            #ManualProduction := 7;
            "MessageDisplay" := 11; //"Station #3 is full. Please empty it"
        END_IF;
        
        //Checking if slide1 and slide2 are simultaneously full
        IF "Slide1Full" AND "Slide2Full" THEN
            #ManualProduction := 8;
            "MessageDisplay" := 12; //"Slides #1 & #2 full. Please collect one of them"
        END_IF;
        
        //This condition is to continue the normal production
        IF ("Slide1Full" = False OR "Slide2Full" = FALSE) AND "Slide3Full" = False THEN
            #ManualProduction := 9;
        END_IF;
        
    7:  //This case is only to empty slide3 when full
        
        IF "ClearSlide3Button" THEN
            "SinkSlide3" := True;
            "Slide3Full" := False;
            "TimerStartStopper" := True;
        END_IF;
        
        IF "LightSensorWorkpieceSlide" = False AND "TimerFinishStopper" = True THEN
            "SinkSlide3" := False;
            "TimerStartStopper" := False;
            "TimerFinishStopper" := False;
            #ManualProduction := 6;
        END_IF;
        
    8:  //This case is only to empty slide1 and/or slide2 when completed/full
        
        IF "ClearSlide1Button" AND "Slide1Full" THEN
            "SinkSlide1" := True;
            "Slide1Full" := False;
            #ManualProduction := 6;
        END_IF;
        
        IF "ClearSlide2Button" AND "Slide2Full" THEN
            "SinkSlide2" := True;
            "Slide2Full" := False;
            #ManualProduction := 6;
        END_IF;
        
    9:
        //Wait until both slide pistons are retracted to restart the cycle
        IF "LimitSwitchPistonRotativeHead1Retracted" AND "LimitSwitchPistonRotativeHead2Retracted" THEN
            #ManualProduction := 0;
        END_IF;
        
    10:
        //This case is only placed when automatic or control button is pressed, to prevent conflicts
        IF #AutomaticProduction = 0 OR #ControlProduction = 0 THEN
            #ManualProduction := 0;
        END_IF;
END_CASE;

//This case is for automatic operation of the production line
CASE #AutomaticProduction OF
    0:
        "WorkpieceBlack" := False;
        "WorkpieceRed" := False;
        "WorkpieceSilver" := False;
        
        //Asking if both orders are 6 units or less, and not 0
        IF ("CounterWorkpieceBlackSlide1" + "CounterWorkpieceRedSlide1"
            + "CounterWorkpieceSilverSlide1") <= 6 AND ("CounterWorkpieceBlackSlide2" +
            "CounterWorkpieceRedSlide2" + "CounterWorkpieceSilverSlide2") <= 6 AND
            ("CounterWorkpieceBlackSlide1" >= 1 OR "CounterWorkpieceRedSlide1" >= 1 OR
            "CounterWorkpieceSilverSlide1" >= 1 OR
            "CounterWorkpieceBlackSlide2" >= 1 OR "CounterWorkpieceRedSlide2" >= 1 OR
            "CounterWorkpieceSilverSlide2" >= 1) THEN
        
            
            //When this button is pressed, it will start the whole automatic process...
            //... and this case won't be 0 until the full process is finished
            IF "AutomaticButton" AND "LightSensorWorkpieceSlide" = False THEN
                #AutomaticProduction := 1;
            END_IF;
        END_IF;
        
        //If manual process is active, then this case will be sent to  #7...
        //... to prevent conflicts
        IF #ManualProduction >= 1 THEN
            #AutomaticProduction := 7;
        END_IF;
        
    1: //This is case is to insert a workpiece in the following order:...
       //BlackSlide1, RedSlide1, SilverSlide1, BlackSlide2, RedSlide2, SilverSlide2
      
       //Some conditions are set to false/default position
        "Conveyor" := False;
        "PistonLinearHeadExtend" := True;
        "PistonLinearHeadRetract" := False;
        "PistonRotativeHead1Extend" := False;
        "PistonRotativeHead1Retract" := True;
        "PistonRotativeHead2Extend" := False;
        "PistonRotativeHead2Retract" := True;
        "BlackInPlay" := False;
        "RedInPlay" := False;
        "SilverInPlay" := False;
        "TimerStartStopper" := False;
        "TimerFinishStopper" := False;
        
        IF "CounterWorkpieceBlackSlide1" >= 1 THEN
            "WorkpieceBlack" := True;
            IF "LightSensorWorkpiece" THEN
                #AutomaticProduction := 2;
                "WorkpieceBlack" := False;
            END_IF;
        ELSIF "CounterWorkpieceRedSlide1" >= 1 THEN
            "WorkpieceRed" := True;
            IF "LightSensorWorkpiece" THEN
                #AutomaticProduction := 2;
                "WorkpieceRed" := False;
            END_IF;
        ELSIF "CounterWorkpieceSilverSlide1" >= 1 THEN
            "WorkpieceSilver" := True;
            IF "LightSensorWorkpiece" THEN
                #AutomaticProduction := 2;
                "WorkpieceSilver" := False;
            END_IF;
        ELSIF "CounterWorkpieceBlackSlide2" >= 1 THEN
            "WorkpieceBlack" := True;
            IF "LightSensorWorkpiece" THEN
                #AutomaticProduction := 2;
                "WorkpieceBlack" := False;
            END_IF;
        ELSIF "CounterWorkpieceRedSlide2" >= 1 THEN
            "WorkpieceRed" := True;
            IF "LightSensorWorkpiece" THEN
                #AutomaticProduction := 2;
                "WorkpieceRed" := False;
            END_IF;
        ELSIF "CounterWorkpieceSilverSlide2" >= 1 THEN
            "WorkpieceSilver" := True;
            IF "LightSensorWorkpiece" THEN
                #AutomaticProduction := 2;
                "WorkpieceSilver" := False;
            END_IF;
        END_IF;
        
        "MessageDisplay" := 1; //"Automatic Mode On"
    
    2:  //This case is to retract linear piston, and activate the...
        //...respective rotative piston
        
        "Conveyor" := True; //This is to active conveyor
        "SinkSlide1" := False; //To deactivate sink, using this line in case 0 or 1 causes bugs
        "SinkSlide2" := False; //To deactivate sink, using this line in case 0 or 1 causes bugs
        "SinkSlide3" := False; //To deactivate sink, using this line in case 0 or 1 causes bugs
        
        //This IF is to turn off the conveyor until the stopper is retracted
        IF ("LightSensorWorkpieceBlack1" OR "LightSensorWorkpieceRed2" OR "LightSensorWorkpieceSilver3")
            AND "TimerFinishStopper" = False THEN
            "Conveyor" := False;
        ELSE
            "Conveyor" := True;
        END_IF;
        
        //When Black workpiece is detected 
        IF "LightSensorWorkpieceBlack1" THEN
            "BlackInPlay" := True;
            "TimerStartStopper" := True;
            IF "TimerFinishStopper" THEN //This is to delay the stopper a bit before opening
                "PistonLinearHeadExtend" := False;
                "PistonLinearHeadRetract" := True;
                IF "CounterWorkpieceBlackSlide1" > 0 THEN
                    "PistonRotativeHead1Extend" := True;
                    "PistonRotativeHead1Retract" := False;
                ELSIF "CounterWorkpieceBlackSlide2" > 0 THEN
                    "PistonRotativeHead2Extend" := True;
                    "PistonRotativeHead2Retract" := False;
                END_IF;
            END_IF;
        END_IF;
        
        //When Red workpiece is detected 
        IF "LightSensorWorkpieceRed2" THEN
            "RedInPlay" := True;
            "TimerStartStopper" := True;
            IF "TimerFinishStopper" THEN //This is to delay the stopper a bit before opening
                "PistonLinearHeadExtend" := False;
                "PistonLinearHeadRetract" := True;
                IF "CounterWorkpieceRedSlide1" > 0 THEN
                    "PistonRotativeHead1Extend" := True;
                    "PistonRotativeHead1Retract" := False;
                ELSIF "CounterWorkpieceRedSlide2" > 0 THEN
                    "PistonRotativeHead2Extend" := True;
                    "PistonRotativeHead2Retract" := False;
                END_IF;
            END_IF;
        END_IF;
        
        //When Silver workpiece is detected 
        IF "LightSensorWorkpieceSilver3" THEN
            "SilverInPlay" := True;
            "TimerStartStopper" := True;
            IF "TimerFinishStopper" THEN //This is to delay the stopper a bit before opening
                "PistonLinearHeadExtend" := False;
                "PistonLinearHeadRetract" := True;
                IF "CounterWorkpieceSilverSlide1" > 0 THEN
                    "PistonRotativeHead1Extend" := True;
                    "PistonRotativeHead1Retract" := False;
                ELSIF "CounterWorkpieceSilverSlide2" > 0 THEN
                    "PistonRotativeHead2Extend" := True;
                    "PistonRotativeHead2Retract" := False;
                END_IF;
            END_IF;
        END_IF;
        
        //When any workpiece goes through any slide
        IF "LightSensorWorkpieceSlide" THEN
            //If piston for slide1 is active
            IF "LimitSwitchPistonRotativeHead1Extended" THEN
                
                //Substract the corresponding colour counter in slide1
                IF "BlackInPlay" THEN     //Checks if black workpiece was placed
                    "CounterWorkpieceBlackSlide1" := "CounterWorkpieceBlackSlide1" - 1;
                ELSIF "RedInPlay" THEN    //Checks if red workpiece was placed
                    "CounterWorkpieceRedSlide1" := "CounterWorkpieceRedSlide1" - 1;
                ELSIF "SilverInPlay" THEN //Checks if silver workpiece was placed
                    "CounterWorkpieceSilverSlide1" := "CounterWorkpieceSilverSlide1" - 1;
                END_IF;
                
                //If piston for slide2 is active
            ELSIF "LimitSwitchPistonRotativeHead2Extended" THEN
                
                //Substract the corresponding colour counter in slide2
                IF "BlackInPlay" THEN
                    "CounterWorkpieceBlackSlide2" := "CounterWorkpieceBlackSlide2" - 1;
                ELSIF "RedInPlay" THEN
                    "CounterWorkpieceRedSlide2" := "CounterWorkpieceRedSlide2" - 1;
                ELSIF "SilverInPlay" THEN
                    "CounterWorkpieceSilverSlide2" := "CounterWorkpieceSilverSlide2" - 1;
                END_IF;
            END_IF;
            
            #AutomaticProduction := 3;
        END_IF;
    3: //This will retract both rotative pistons to its original position...
        //... and check if any slide is completed/full
        
        //Checking if Slide1 was active
        IF "LimitSwitchPistonRotativeHead1Extended" THEN
            //Checking if Slide1 order is completed/full
            IF ("CounterWorkpieceBlackSlide1" = 0 AND
                "CounterWorkpieceRedSlide1" = 0 AND
                "CounterWorkpieceSilverSlide1" = 0) OR "TimerSlideFullFinish" THEN
                "Slide1Full" := True;
            END_IF;
            
            //Resetting piston for Slide1
            "PistonRotativeHead1Extend" := False;
            "PistonRotativeHead1Retract" := True;
            #AutomaticProduction := 4;
        END_IF;
        
        //Checking if slide2 was active
        IF "LimitSwitchPistonRotativeHead2Extended" THEN
            //Checking if Slide2 order is completed/full
            IF ("CounterWorkpieceBlackSlide2" = 0 AND
                "CounterWorkpieceRedSlide2" = 0 AND
                "CounterWorkpieceSilverSlide2" = 0) OR "TimerSlideFullFinish" THEN
                "Slide2Full" := True;
            END_IF;
            
            //Resetting piston for Slide2
            "PistonRotativeHead2Extend" := False;
            "PistonRotativeHead2Retract" := True;
            #AutomaticProduction := 4;
        END_IF;
        
        //To reset linear piston
        "PistonLinearHeadExtend" := True;
        "PistonLinearHeadRetract" := False;
        
    4:  //This case is to verify  if slide 1 got full
        
        IF "Slide1Full" AND "LightSensorWorkpieceSlide" THEN
            #AutomaticProduction := 5;
        END_IF;
        
        //This condition is to continue the normal production
        IF ("Slide1Full" = False OR "Slide2Full" = FALSE) AND "Slide3Full" = False THEN
            #AutomaticProduction := 6;
        END_IF;
        
    5:  //This case is only to empty slide1 when full
        IF "LightSensorWorkpieceSlide" = False THEN
            #AutomaticProduction := 6;
        ELSIF "ClearSlide1Button" AND "Slide1Full" THEN
            "SinkSlide1" := True;
            "Slide1Full" := False;
            #AutomaticProduction := 6;
        END_IF;
        
    6:
        //Wait until both slide pistons are retracted to restart the cycle
        IF "LimitSwitchPistonRotativeHead1Retracted" AND "LimitSwitchPistonRotativeHead2Retracted" THEN
            
            //Asking if both slides are completed to reset the production,...
            //...otherwise it will continue inserting workpieces automatically
            IF "CounterWorkpieceBlackSlide1" = 0 AND "CounterWorkpieceRedSlide1" = 0 AND
                "CounterWorkpieceSilverSlide1" = 0 AND "CounterWorkpieceBlackSlide2" = 0 AND
                "CounterWorkpieceRedSlide2" = 0 AND "CounterWorkpieceSilverSlide2" = 0 THEN
                #AutomaticProduction := 0;
            ELSE
                #AutomaticProduction := 1;
            END_IF;
        END_IF;
        
    7:
        //This case is only placed when manual or control process is active, to prevent conflicts
        IF #ManualProduction = 0 OR #ControlProduction = 0 THEN
            #AutomaticProduction := 0;
        END_IF;
        
END_CASE;

//This case is for control operation of the production line
CASE #ControlProduction OF
    0:
        //Asking if both orders are 6 units or less, and not 0
        IF ("CounterWorkpieceBlackSlide1" + "CounterWorkpieceRedSlide1"
            + "CounterWorkpieceSilverSlide1") <= 6 AND ("CounterWorkpieceBlackSlide2" +
            "CounterWorkpieceRedSlide2" + "CounterWorkpieceSilverSlide2") <= 6 AND
            ("CounterWorkpieceBlackSlide1" >= 1 OR "CounterWorkpieceRedSlide1" >= 1 OR
            "CounterWorkpieceSilverSlide1" >= 1 OR
            "CounterWorkpieceBlackSlide2" >= 1 OR "CounterWorkpieceRedSlide2" >= 1 OR
            "CounterWorkpieceSilverSlide2" >= 1) THEN
            
            
            //When this button is pressed, it will send to the next case that enables...
            //... the buttons for control
            IF "ControlButton" AND "LightSensorWorkpieceSlide" = False THEN
                #ControlProduction := 1;
                "MessageDisplay" := 18; //"Control Mode On"
            END_IF;
        END_IF;
        
        //If manual or automatic process is active, then this case will be sent to  #7...
        //... to prevent conflicts
        IF #AutomaticProduction >= 1 OR #ManualProduction >= 1 THEN
            #ControlProduction := 11;
        END_IF;
        
    1: //This case will run until a workpiece button is pressed
        "SinkSlide1" := False; //To deactivate sink, using this line in case 0 causes bugs
        "SinkSlide2" := False; //To deactivate sink, using this line in case 0 causes bugs
        "SinkSlide3" := False; //To deactivate sink, using this line in case 0 causes bugs
        
        "MessageDisplay" := 18; //"Control Mode On"
        
        IF "ConveyorButton" THEN
            "Conveyor" := True;
            "MessageDisplay" := 19; //"Conveyor activated"
        ELSE
            "Conveyor" := False;
        END_IF;
        
        IF "BlackControlButton" THEN
            "WorkpieceBlack" := True;
        END_IF;
        
        IF "RedControlButton" THEN
            "WorkpieceRed" := True;
        END_IF;
        
        IF "SilverControlButton" THEN
            "WorkpieceSilver" := True;
        END_IF;
        
        IF "StopperOnButton" THEN
            "PistonLinearHeadExtend" := True;
            "PistonLinearHeadRetract" := False;
            "MessageDisplay" := 20; //"Stopper extended"
        END_IF;
        
        IF "StopperOffButton" THEN
            "PistonLinearHeadExtend" := False;
            "PistonLinearHeadRetract" := True;
            "MessageDisplay" := 21; //"Stopper retracted"
        END_IF;
        
        IF "Piston1OnButton" THEN
            "PistonRotativeHead1Extend" := True;
            "PistonRotativeHead1Retract" := False;
            "PistonRotativeHead2Extend" := False;
            "PistonRotativeHead2Retract" := True;
            "MessageDisplay" := 22; //"Piston #1 extended"
        END_IF;
        
        IF "Piston1OffButton" THEN
            "PistonRotativeHead1Extend" := False;
            "PistonRotativeHead1Retract" := True;
            "MessageDisplay" := 23; //"Piston #1 retracted"
        END_IF;
        
        IF "Piston2OnButton" THEN
            "PistonRotativeHead2Extend" := True;
            "PistonRotativeHead2Retract" := False;
            "PistonRotativeHead1Extend" := False;
            "PistonRotativeHead1Retract" := True;
            "MessageDisplay" := 24; //"Piston #2 extended"
        END_IF;
        
        IF "Piston2OffButton" THEN
            "PistonRotativeHead2Extend" := False;
            "PistonRotativeHead2Retract" := True;
            "MessageDisplay" := 25; //"Piston #2 retracted"
        END_IF;
        
        
        //If a workpiece is detected, case will be sent to 2
        IF "LightSensorWorkpiece" THEN
            "WorkpieceBlack" := False;
            "WorkpieceRed" := False;
            "WorkpieceSilver" := False;
            "MessageDisplay" := 16; //"Workpiece detected"
            #ControlProduction := 2;
        END_IF;
        
    2:  //This case is exactly like the previous one, but workpiece buttons are disabled
        //This case is active until a workpiece touches the slide sensor
        
        //Checking which color workpiece has been inserted
        IF "LightSensorWorkpieceBlack1" THEN
            "BlackInPlay" := True;
        ELSIF "LightSensorWorkpieceRed2" THEN
            "RedInPlay" := True;
        ELSIF "LightSensorWorkpieceSilver3" THEN
            "SilverInPlay" := True;
        END_IF;
        
        IF "ConveyorButton" THEN
            "Conveyor" := True;
            "MessageDisplay" := 19; //"Conveyor activated"
        ELSE
            "Conveyor" := False;
        END_IF;
        
        IF "StopperOnButton" THEN
            "PistonLinearHeadExtend" := True;
            "PistonLinearHeadRetract" := False;
            "MessageDisplay" := 20; //"Stopper extended"
        END_IF;
        
        IF "StopperOffButton" THEN
            "PistonLinearHeadExtend" := False;
            "PistonLinearHeadRetract" := True;
            "MessageDisplay" := 21; //"Stopper retracted"
        END_IF;
        
        IF "Piston1OnButton" THEN
            "PistonRotativeHead1Extend" := True;
            "PistonRotativeHead1Retract" := False;
            "PistonRotativeHead2Extend" := False;
            "PistonRotativeHead2Retract" := True;
            "MessageDisplay" := 22; //"Piston #1 extended"
        END_IF;
        
        IF "Piston1OffButton" THEN
            "PistonRotativeHead1Extend" := False;
            "PistonRotativeHead1Retract" := True;
            "MessageDisplay" := 23; //"Piston #1 retracted"
        END_IF;
        
        IF "Piston2OnButton" THEN
            "PistonRotativeHead2Extend" := True;
            "PistonRotativeHead2Retract" := False;
            "PistonRotativeHead1Extend" := False;
            "PistonRotativeHead1Retract" := True;
            "MessageDisplay" := 24; //"Piston #2 extended"
        END_IF;
        
        IF "Piston2OffButton" THEN
            "PistonRotativeHead2Extend" := False;
            "PistonRotativeHead2Retract" := True;
            "MessageDisplay" := 25; //"Piston #2 retracted"
        END_IF;
        
        //When sensor slide is triggered, case will be sent to 3
        IF "LightSensorWorkpieceSlide" THEN
            "Conveyor" := False;
            "PistonLinearHeadExtend" := True;
            "PistonLinearHeadRetract" := False;
            #ControlProduction := 3;
        END_IF;
        
        
    3:   //First check if the workpiece is placed in the wrong order
        
        IF    "LimitSwitchPistonRotativeHead1Extended" AND "BlackInPlay" AND "CounterWorkpieceBlackSlide1" <= 0 THEN
            #ControlProduction := 4;
        ELSIF "LimitSwitchPistonRotativeHead1Extended" AND "RedInPlay" AND "CounterWorkpieceRedSlide1" <= 0 THEN
            #ControlProduction := 4;
        ELSIF "LimitSwitchPistonRotativeHead1Extended" AND "SilverInPlay" AND "CounterWorkpieceSilverSlide1" <= 0 THEN
            #ControlProduction := 4;
        ELSIF "LimitSwitchPistonRotativeHead2Extended" AND "BlackInPlay" AND "CounterWorkpieceBlackSlide2" <= 0 THEN
            #ControlProduction := 4;
        ELSIF "LimitSwitchPistonRotativeHead2Extended" AND "RedInPlay" AND "CounterWorkpieceRedSlide2" <= 0 THEN
            #ControlProduction := 4;
        ELSIF "LimitSwitchPistonRotativeHead2Extended" AND "SilverInPlay" AND "CounterWorkpieceSilverSlide2" <= 0 THEN
            #ControlProduction := 4;
        ELSE
            #ControlProduction := 5;
        END_IF;
                
    4: //This case is activated when a workpiece is in the wrong order
    
        IF "LimitSwitchPistonRotativeHead1Extended" THEN
            "MessageDisplay" := 26; //"ERROR: Wrong workpiece in Slide1. Please empty it"
            IF "ClearSlide1Button" THEN
                "SinkSlide1" := True;
                "CountersSlide1Visibility" := False;
                #Slide1Counters := 0; //Reset Slide1Counters case
                "PistonRotativeHead1Extend" := False;
                "PistonRotativeHead1Retract" := True;
                "CounterWorkpieceBlackSlide1" := 0;
                "CounterStaticWorkpieceRedSlide1" := 0;
                "CounterStaticWorkpieceSilverSlide1" := 0;
                #ControlProduction := 0;
            END_IF;
        END_IF;
        
        IF "LimitSwitchPistonRotativeHead2Extended" THEN
            "MessageDisplay" := 27; //"ERROR: Wrong workpiece in Slide2. Please empty it"
            IF "ClearSlide2Button" THEN
                "SinkSlide2" := True;
                "CountersSlide2Visibility" := False;
                #Slide2Counters := 0; //Reset Slide2Counters case
                "PistonRotativeHead2Extend" := False;
                "PistonRotativeHead2Retract" := True;
                "CounterWorkpieceBlackSlide2" := 0;
                "CounterStaticWorkpieceRedSlide2" := 0;
                "CounterStaticWorkpieceSilverSlide2" := 0;
                #ControlProduction := 0;
            END_IF;
        END_IF;
                
    5: //This case run when the workpiece goes into the correct order or slide3
        
        //If piston for slide1 is active
        IF "LimitSwitchPistonRotativeHead1Extended" THEN
            
            //Substract the corresponding colour counter in slide1
            IF "BlackInPlay" THEN     //Checks if black workpiece was placed
                "CounterWorkpieceBlackSlide1" := "CounterWorkpieceBlackSlide1" - 1;
            ELSIF "RedInPlay" THEN    //Checks if red workpiece was placed
                "CounterWorkpieceRedSlide1" := "CounterWorkpieceRedSlide1" - 1;
            ELSIF "SilverInPlay" THEN //Checks if silver workpiece was placed
                "CounterWorkpieceSilverSlide1" := "CounterWorkpieceSilverSlide1" - 1;
            END_IF;
            
            //If piston for slide2 is active
        ELSIF "LimitSwitchPistonRotativeHead2Extended" THEN
            
            //Substract the corresponding colour counter in slide2
            IF "BlackInPlay" THEN
                "CounterWorkpieceBlackSlide2" := "CounterWorkpieceBlackSlide2" - 1;
            ELSIF "RedInPlay" THEN
                "CounterWorkpieceRedSlide2" := "CounterWorkpieceRedSlide2" - 1;
            ELSIF "SilverInPlay" THEN
                "CounterWorkpieceSilverSlide2" := "CounterWorkpieceSilverSlide2" - 1;
            END_IF;
        END_IF;
            
            #ControlProduction := 6;
            
        6: //This will retract both rotative pistons to its original position...
            //... and check if any slide is completed/full
            
            //Checking if Slide3 is full
            IF "LimitSwitchPistonRotativeHead1Extended" = False AND
                "LimitSwitchPistonRotativeHead2Extended" = False AND
                "TimerSlideFullFinish" = True THEN
                "Slide3Full" := True;
                #ControlProduction := 7;
            ELSIF "LimitSwitchPistonRotativeHead1Extended" = False AND
                "LimitSwitchPistonRotativeHead2Extended" = False AND
                "LightSensorWorkpieceSlide" = False THEN
                #ControlProduction := 7;
            END_IF;
            
            //Checking if Slide1 was active
            IF "LimitSwitchPistonRotativeHead1Extended" THEN
                //Checking if Slide1 order is completed/full
                IF ("CounterWorkpieceBlackSlide1" = 0 AND
                    "CounterWorkpieceRedSlide1" = 0 AND
                    "CounterWorkpieceSilverSlide1" = 0) OR "TimerSlideFullFinish" THEN
                    "Slide1Full" := True;
                END_IF;
                
                //Resetting piston for Slide1
                "PistonRotativeHead1Extend" := False;
                "PistonRotativeHead1Retract" := True;
                #ControlProduction := 7;
            END_IF;
            
            //Checking if slide2 was active
            IF "LimitSwitchPistonRotativeHead2Extended" THEN
                //Checking if Slide2 order is completed/full
                IF ("CounterWorkpieceBlackSlide2" = 0 AND
                    "CounterWorkpieceRedSlide2" = 0 AND
                    "CounterWorkpieceSilverSlide2" = 0) OR "TimerSlideFullFinish" THEN
                    "Slide2Full" := True;
                END_IF;
                
                //Resetting piston for Slide2
                "PistonRotativeHead2Extend" := False;
                "PistonRotativeHead2Retract" := True;
                #ControlProduction := 7;
            END_IF;
            
            //To reset linear piston
            "PistonLinearHeadExtend" := True;
            "PistonLinearHeadRetract" := False;
            
        7:  //This case is to verify  if slides 1 and 2 are simultaneously full,...
            //or if slide3 is full. If any condition is true it will be sent to case 7 or 8
            
            //Checking if slide3 is full
            IF "Slide3Full" THEN
                #ControlProduction := 8;
                "MessageDisplay" := 11; //"Station #3 is full. Please empty it"
            END_IF;
            
            //Checking if slide1 and slide2 are simultaneously full
            IF "Slide1Full" AND "Slide2Full" THEN
                #ControlProduction := 9;
                "MessageDisplay" := 12; //"Slides #1 & #2 full. Please collect one of them"
            END_IF;
            
            //This condition is to continue the normal production
            IF ("Slide1Full" = False OR "Slide2Full" = FALSE) AND "Slide3Full" = False THEN
                #ControlProduction := 10;
            END_IF;
            
        8:  //This case is only to empty slide3 when full
            
            IF "ClearSlide3Button" THEN
                "SinkSlide3" := True;
                "Slide3Full" := False;
                "TimerStartStopper" := True;
            END_IF;
            
            IF "LightSensorWorkpieceSlide" = False AND "TimerFinishStopper" = True THEN
                "SinkSlide3" := False;
                "TimerStartStopper" := False;
                "TimerFinishStopper" := False;
                #ControlProduction := 7;
            END_IF;
            
        9:  //This case is only to empty slide1 and/or slide2 when completed/full
            
            IF "ClearSlide1Button" AND "Slide1Full" THEN
                "SinkSlide1" := True;
                "Slide1Full" := False;
                #ControlProduction := 7;
            END_IF;
            
            IF "ClearSlide2Button" AND "Slide2Full" THEN
                "SinkSlide2" := True;
                "Slide2Full" := False;
                #ControlProduction := 7;
            END_IF;
            
        10:
            //Wait until both slide pistons are retracted to restart the cycle
            IF "LimitSwitchPistonRotativeHead1Retracted" AND "LimitSwitchPistonRotativeHead2Retracted" THEN
                #ControlProduction := 0;
            END_IF;
            
        11:
            //This case is only placed when automatic or manual button is pressed, to prevent conflicts
            IF #AutomaticProduction = 0 OR #ManualProduction = 0 THEN
                #ControlProduction := 0;
            END_IF;
    END_CASE;

CASE #Slide1Counters OF
    0:
        //Asking if order1 is less or equal to 6, and not 0
        IF ("CounterWorkpieceBlackSlide1" + "CounterWorkpieceRedSlide1"
            + "CounterWorkpieceSilverSlide1") <= 6 AND "LightSensorWorkpieceSlide" = False AND
            ("CounterWorkpieceBlackSlide1" >= 1 OR "CounterWorkpieceRedSlide1" >= 1 OR
            "CounterWorkpieceSilverSlide1" >= 1) THEN
            
            //Asking which buttons are pressed
            IF ("BlackWorkpieceButton" OR "RedWorkpieceButton" OR "SilverWorkpieceButton") OR
                "AutomaticButton" OR "ControlButton" THEN
                "CounterStaticWorkpieceBlackSlide1" := "CounterWorkpieceBlackSlide1";
                "CounterStaticWorkpieceRedSlide1" := "CounterWorkpieceRedSlide1";
                "CounterStaticWorkpieceSilverSlide1" := "CounterWorkpieceSilverSlide1";
                #Slide1Counters := 1;
                "CountersSlide1Visibility" := True;
            END_IF;
        END_IF;
    1:
        IF "Slide1Full" THEN
            #Slide1Counters := 2;
        END_IF;
        
    2: IF "ClearSlide1Button" AND (#ManualProduction = 0 OR #ManualProduction = 8) THEN
            "CountersSlide1Visibility" := False;
            #Slide1Counters := 0;
        END_IF;
END_CASE;

CASE #Slide2Counters OF
    0:
        //Asking if order1 is less or equal to 6, and not 0
        IF ("CounterWorkpieceBlackSlide2" + "CounterWorkpieceRedSlide2"
            + "CounterWorkpieceSilverSlide2") <= 6 AND "LightSensorWorkpieceSlide" = False AND
            ("CounterWorkpieceBlackSlide2" >= 1 OR "CounterWorkpieceRedSlide2" >= 1 OR
            "CounterWorkpieceSilverSlide2" >= 1) THEN
            
            //Asking which buttons are pressed
            IF ("BlackWorkpieceButton" OR "RedWorkpieceButton" OR "SilverWorkpieceButton") OR
                "AutomaticButton" OR "ControlButton" THEN
                "CounterStaticWorkpieceBlackSlide2" := "CounterWorkpieceBlackSlide2";
                "CounterStaticWorkpieceRedSlide2" := "CounterWorkpieceRedSlide2";
                "CounterStaticWorkpieceSilverSlide2" := "CounterWorkpieceSilverSlide2";
                "CountersSlide2Visibility" := True;
                #Slide2Counters := 1;
            END_IF;
        END_IF;
    1:
        IF "Slide2Full" THEN
            #Slide2Counters := 2;
        END_IF;
        
    2:
        IF "ClearSlide2Button" AND (#ManualProduction = 0 OR #ManualProduction = 8) THEN
            "CountersSlide2Visibility" := False;
            #Slide2Counters := 0;
        END_IF;
        
END_CASE;

//This case is to control the green LED in the TowerLight on the NX Model
CASE #GreenLED OF
    0: //Green starts off until slide1 is full
        "GreenLight" := False;
        
        IF "Slide1Full" THEN
            "SinkGreenLight" := False;
            #GreenLED := 1;
        END_IF;
    1: //To turn on green light
        "GreenLight" := True;
        #GreenLED := 2;
    2: //Wait until slide is cleared, causing green light to turn off
        IF "GreenLightSensor" THEN
            "GreenLight" := False;
        END_IF;
        
        IF "Slide1Full" = False THEN
            "SinkGreenLight" := True;
            #GreenLED := 0;
        END_IF;
        
END_CASE;

//This case is to control the yellow LED in the TowerLight on the NX Model
CASE #YellowLED OF
    0: //Yellow starts off until slide2 is full
        "YellowLight" := False;
        
        IF "Slide2Full" THEN
            "SinkYellowLight" := False;
            #YellowLED := 1;
        END_IF;
    1: //To turn on yellow light
        "YellowLight" := True;
        #YellowLED := 2;
    2: //Wait until slide is cleared, causing yellow light to turn off
        IF "YellowLightSensor" THEN
            "YellowLight" := False;
        END_IF;
        
        IF "Slide2Full" = False THEN
            "SinkYellowLight" := True;
            #YellowLED := 0;
        END_IF;
        
END_CASE;

//This case is to control the red LED in the TowerLight on the NX Model
CASE #RedLED OF
    0: //Red starts off until slide3 is full
        "RedLight" := False;
        
        IF "Slide3Full" THEN
            "SinkRedLight" := False;
            #RedLED := 1;
        END_IF;
    1: //To turn on red light
        "RedLight" := True;
        #RedLED := 2;
    2: //Wait until slide is cleared, causing red light to turn off
        IF "RedLightSensor" THEN
            "RedLight" := False;
        END_IF;
        
        IF "Slide3Full" = False THEN
            "SinkRedLight" := True;
            #RedLED := 0;
        END_IF;
        
END_CASE;

//This timer is to detect when a slide is full, activating...
//...when the slide sensor is true for more than 2 seconds
IF "LightSensorWorkpieceSlide" THEN
    "TimerSlideFullStart" := True;
    "IEC_Timer_0_DB".TON(IN := "TimerSlideFullStart",
                           PT := t#2000ms,
                           Q => "TimerSlideFullFinish");
  ELSE
      RESET_TIMER("IEC_Timer_0_DB");
    "TimerSlideFullStart" := False;
END_IF;

//This timer is to delay the stopper retract
"IEC_Timer_0_DB_1".TON(IN := "TimerStartStopper",
                     PT := t#3000ms,
                     Q => "TimerFinishStopper");
