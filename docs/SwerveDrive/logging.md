# Logging Workflow

## Overview

This section will describe how [Advantage Kit](https://github.com/Mechanical-Advantage/AdvantageKit) is integrated with the swerve code. The general approach will be to have an intermediate states class for each subsystem. This class will store the states of the subsystem(like the drive speed for the module subsystem). At each call of the periodic function, the Advantage Kit Logger will take an instance of the states class to automatically log values from it. For a more detailed explanation, look at the [Advantage Kit Documentation](https://github.com/Mechanical-Advantage/AdvantageKit/blob/main/README.md). These pages might be especially helpful:

- [Data Flow](https://github.com/Mechanical-Advantage/AdvantageKit/blob/main/docs/DATA-FLOW.md)
- [Recording Inputs](https://github.com/Mechanical-Advantage/AdvantageKit/blob/main/docs/RECORDING-INPUTS.md)
!!! note
	Before reading this section, please make sure you have a strong understanding of object oriented programming, and the command-based programming paradigm. If you are new, please read the [WPI Documentation](https://docs.wpilib.org/en/stable/docs/software/commandbased/index.html) for info on command-based programming.
## Installation
Install the Advantage Kit library like any third party library by right clicking on `build.gradle` and clicking `Manage Vendor Libraries`. Once you click on online install, paste the following link to download the json file:
```
https://github.com/Mechanical-Advantage/AdvantageKit/releases/latest/download/AdvantageKit.json
```

!!! note
	This link may be out of date. If you get a compilation error, go to the [documentation](https://github.com/Mechanical-Advantage/AdvantageKit/blob/main/docs/INSTALLATION.md) for the latest link to the json file.

Now open the `build.gradle` file. Make sure the repositories block in the file looks like this:
```
repositories {

    maven {
        url = uri("https://maven.pkg.github.com/Mechanical-Advantage/AdvantageKit")
        credentials {
            username = "Mechanical-Advantage-Bot"
            password = "\u0067\u0068\u0070\u005f\u006e\u0056\u0051\u006a\u0055\u004f\u004c\u0061\u0079\u0066\u006e\u0078\u006e\u0037\u0051\u0049\u0054\u0042\u0032\u004c\u004a\u006d\u0055\u0070\u0073\u0031\u006d\u0037\u004c\u005a\u0030\u0076\u0062\u0070\u0063\u0051"
        }
    }
    mavenLocal()
    mavenCentral()

	}
```
 If you do not see a repositories block, make one under the `includeDesktopSupport` line and format it as shown above.
 

## Setup

Most of the setup for Advantage Kit will take place in the Robot.java file. The first thing to do in Robot.java is to replace the parent class of the robot with `LoggedRobot` instead of `TimedRobot`. 
```java
public class Robot extends LoggedRobot{
...
}
```
Then, the code here can be copy-pasted into Robot.java to finish the setup. It may change from year to year, but generally takes care of the boilerplate to initiate the logging system. If any errors are present, make sure to check the documentation linked above for changes. 
```java
  @Override
  public void robotInit() {
    Logger.recordMetadata("ProjectName", "MyProject"); // Set a metadata value
    if (isReal()) {
        Logger.addDataReceiver(new WPILOGWriter()); // Log to a USB stick
        Logger.addDataReceiver(new NT4Publisher()); // Publish data to NetworkTables
        new PowerDistribution(1, ModuleType.kAutomatic); // Enables power distribution logging
    } else {
        Logger.addDataReceiver(new NT4Publisher()); // Publish data to NetworkTables
    }

    // Logger.getInstance().disableDeterministicTimestamps() // See "Deterministic Timestamps" in the "[Understanding Data Flow" page
    Logger.start();
}
```

Make sure that classes like `Logger`, `NT4Publisher`, and `WPILOGWriter` are imported from `org.littletonrobotics` and not `com.ctre` or `java.util`. Look at the [2024 Robot Code](https://github.com/frc3748/2024-Robot-Code/blob/master/src/main/java/frc/robot/Robot.java) for examples of import statements and how Advantage Kit might be integrated with the rest of the robot code. 
!!! Note
	Advantage Kit can be configured to omit logs from the Network Tables or the USB stick. To do this, comment out the appropriate `Logger.addDataReceiver()` lines.

## The Approach to Logging

The way Advantage Kit differs to traditional logging methods is that with this library, all values that could be useful get logged. Instead of putting certain values to the network tables and having all of the logs as one list that gets deleted upon reboot, Advantage Kit enables organization of logged values by Subsystem and saves logs to a USB stick. For more details, read [this](https://github.com/Mechanical-Advantage/AdvantageKit/blob/main/docs/WHAT-IS-ADVANTAGEKIT.md) page.

In order to organize logged values by subsystem, subsystem that is going to be logged will have a`states` attribute that stores the current states of the subsystem. For instance, for a drivetrain subsystem, the `states` object could store the current chassis speed, or the angle of the chassis with respect to the field. Every time the `periodic()` function is called, an `updateStates()` function will be called that reads the newest values from all the sensors and stores them in the `states` object. This object will then be passed to Advantage Kits `Logger` class so that the values can be stored on a USB and/or logged to Smart Dashboard.
## Implementing Advantage Kit

Each subsystem that is going to be logged is going to have a corresponding interface that defines which values are going to be logged through a subsystem. It will also contain an interface method that is going to be updating those values(usually called `updateStates()` as described [above](#the-approach-to-logging)).  This class defined in the interface will  be decorated with the `@AutoLog` decorator which tells the Advantage Kit Logger to use the variable names in the class as the titles for the logged values(you don't need to worry too much about how this is done).

The subsystems in the [2024 Robot Code](https://github.com/frc3748/2024-Robot-Code/tree/master/src/main/java/frc/robot/Subsystems) have several examples of this. Shown below is the interface paired with the drivetrain subsystem.

```java
public interface IDriveIO {

    @AutoLog
    public class DriveStates{
        double gyroAngleDeg;
        double[] odomPoses = {0,0,0};
        double[] chassisSpeeds = {0,0,0};
        double[] targetSpeeds = {0,0,0};
        double[] targetAutoSpeeds = {0, 0, 0};
        boolean recievedNewControls;
    }

    public void updateDriveStates(DriveStatesAutoLogged states);
    /* 
    The body of this method will be defined in 
    the drivetrain subsystem. It will update the 
    values in the states parameter.
    */
}
```

Notice the class that is decorated with `@AutoLog` and the `updateDriveStates` method. Pay careful attention to how the `updateDriveStates` method takes in `DriveStatesAutoLogged` and not the `DriveStates` class. The `DriveStatesAutoLogged` class is automatically generated by Advantage Kit because the `DriveStates` class was decorated with the `@AutoLog` decorator. If you have errors when trying to get the `AutoLogged` classes, try refreshing the java workspace in VS Code or reopening the editor.

The drivetrain subsystem will implement this interface (along with any other interfaces necessary for the subsystem):
```java
public class Drive implements IDriveIO, Subsystem, Sendable{
    DriveStatesAutoLogged states =  new DriveStatesAutoLogged();
    ...
}
```

Also notice that the subsystem has an instance of the `DriveStatesAutoLogged` class as described [above](#the-approach-to-logging).

The drivetrain subsystem will also define the `updateDriveStates` method that is required by the `IDriveIO` interface:
```java
    @Override
    public void updateDriveStates(DriveStatesAutoLogged states){
        states.gyroAngleDeg = gyro.getAngle().getDegrees();
        ChassisSpeeds speeds = getChassisSpeeds();
        states.chassisSpeeds[0] = speeds.vxMetersPerSecond;
        states.chassisSpeeds[1] = speeds.vyMetersPerSecond;
        states.chassisSpeeds[2] = speeds.omegaRadiansPerSecond;
        double[] _odomPose = {
            poseEstimator.getEstimatedPosition().getX(),
            poseEstimator.getEstimatedPosition().getY(),
            poseEstimator.getEstimatedPosition().getRotation().getDegrees()
        };
        states.odomPoses = _odomPose;
        states.recievedNewControls = DriverStation.isNewControlData();
    }
```
Again, this is where all the newest values from the sensors are updated on the states object. This method is called repeatedly in the periodic() method of the subsystem and the updated states are passed to the logger through the `Logger.processInputs()` method:
```java
    @Override
    public void periodic(){
        updateDriveStates(states);
        Logger.processInputs("DriveTrain", states);
        ...
    }
```

## Some Final Notes on Using Advantage Kit

Advantage Kit aims to log every useful sensor value on the robot. As such, any useful values will theoretically be stored in the states object. Since the states object is repeatedly updated with the latest sensor values, it would be redundant to get the values from the sensor again in other parts of the code. So, ideally, all other parts of the code that need sensor values will get it from the states object. Here is an example:
```java
@Override
public void initSendable(SendableBuilder builder) {
	builder.addDoubleProperty("Angle", () -> states.odomPoses[2], null);
	builder.addDoubleProperty("PoseX", () -> states.odomPoses[0], null);
	builder.addDoubleProperty("PoseY", () -> states.odomPoses[1], null);
}
```
Notice that instead of calling `poseEstimator.getEstimatedPosition()` to get the odometry values, this block uses the states object. This is valid because during each `periodic()` call, the `updateDriveStates()` method fills the `states` object with the latest odometry values.

Using the `states` object frees up the CAN bus because you avoid duplicate calls to the sensors for data. Therefore, the ideal way to use Advantage Kit would be to log every useful sensor value and then use the states object in any other place where sensor data is necessary. This is not a requirement, but it is a **highly recommended** optimization that can be implemented if CAN usage ends up being an issue.