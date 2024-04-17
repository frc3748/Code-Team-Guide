# The Swerve Modules

## Overview

This section will describe code involved in controlling the modules. This will be done through a module subsystem that is not concerned with robot-centric or field-centric control. It's sole purpose will be to take in input from higher-level classes regarding module angles and speeds and enforce those inputs as efficiently as possible. It will also have methods to get feedback information like the distance travelled by the modules, their speeds, angles, etc.
