General Code Structer
=====

Finite State Machines
------------
Finite State Machines (FSM) are often used while programming in order to allow for more complex series of actions.
 This is especially useful when one needs multiple tasks to run at the same time, because it allows for tasks to depend on each other’s execution in a non-linear fashion.

In Orbit FTC, we use state machines to define each system's and the whole robot's current state.


Implementation
----------------
First, we need to define all of our robot's states:
^^^^^^^^^^^^^^
.. code-block:: java

   public enum RobotState {
    TRAVEL, INTAKE, CLAWINTAKE, DEPLETE
 }

Then, to define the wanted state from the driver we need to make the following function:
^^^^^^^^^^^^^^
.. code-block:: java
    
    private static RobotState getState(Gamepad gamepad) {
        return gamepad.dpad_up ? RobotState.TRAVEL
                : gamepad.dpad_right ? RobotState.INTAKE
                        : gamepad.dpad_left ? RobotState.CLAWINTAKE : lastState;
    }

After we know the wanted state, now we need to create a function that will run periodically:
^^^^^^^^^^^^^^
..note::
     
     Disclaimer this code is just a simplified version of 14029's powerplay code. it will not work on the actual robot
.. code-block:: java
      
      private static void setSubsystemToState(Gamepad gamepad1, Gamepad gamepad2, Telemetry telemetry) {
            GlobalData.robotState = getState(gamepad1);
            switch (GlobalData.robotState) {
                case TRAVEL:
                    intakeState = IntakeState.STOP;
                    elevatorState = ElevatorStates.GROUND;
                    armState = ArmState.BACK;
                    clawState = ClawState.OPEN;
                    break;
                case INTAKE:
                        elevatorState = ElevatorStates.INTAKE;
                        intakeState = IntakeState.COLLECT;
                        armState = ArmState.BACK;
                        clawState = ClawState.OPEN;
                    break;
                case CLAWINTAKE:
                        intakeState = IntakeState.STOP;
                        elevatorState = ElevatorStates.INTAKE;
                        armState = ArmState.BACK;
                        clawState = ClawState.OPEN;
                    break;
                case DEPLETE:
                    intakeState = IntakeState.STOP;
                    clawState = ClawState.OPEN;
                    armState = ArmState.FRONT;
                    elevatorState = ElevatorStates.DEPLETE;
                    break;
            }

        Intake.operate(intakeState);
        Claw.operate(clawState);
        Arm.operate(armState);
        Elevator.operate(elevatorState)
        lastState = GlobalData.robotState;
    }

generic intake code example:
^^^^^^^^^^^^^^
.. code-block:: java

   public enum IntakeState {
    COLLECT, STOP, DEPLETE
    }






.. code-block:: java

public class Intake {
    public static final DcMotor motors[] = new DcMotor[2];
    private static float power;

    public static void init(HardwareMap hardwareMap) {

        motors[0] = hardwareMap.get(DcMotor.class, "IntakeR");
        motors[1] = hardwareMap.get(DcMotor.class, "IntakeL");

        motors[1].setDirection(DcMotorSimple.Direction.REVERSE);
        for (final DcMotor motor : motors) {
            motor.setZeroPowerBehavior(DcMotor.ZeroPowerBehavior.BRAKE);
        }
    }

    public static void operate(IntakeState state) {
        switch (state) {
            case COLLECT:
                power = IntakeConstants.intakePower;
                break;
            case STOP:
                power = 0;
                break;
            case DEPLETE:
                power = IntakeConstants.depletePower;
                break;
        }

        for (final DcMotor motor : motors)
            motor.setPower(power);
        }
    }


