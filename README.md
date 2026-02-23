Full Code of Simon Says Project Below:
 % Pin setup
buttonPins = ["D2", "D4", "D6", "D8"]; % Button input pins
ledPins = ["D3", "D5", "D7", "D9"];	% LED output pins
 
roundsToWin = 10;              % Total rounds to win
roundCounter = 0;              % Tracks the current round
timeLimit = 2;                 % Time limit (in seconds) for button press
gameStarted = false;           % Game state flag
 
% Initialize Arduino
a = arduino('COM5','Uno');
 
% Configure pins
for i = 1:4
	configurePin(a, buttonPins(i), 'Pullup');   	% Buttons as input
	configurePin(a, ledPins(i), 'DigitalOutput');  % LEDs as output
end
 
% Main game loop
while true
	if ~gameStarted
    	% Start game and generate sequence
    	buttonSequence = randi([1, 4], 1, roundsToWin); % Generate random sequence
    	roundCounter = 0;
    	flashAllLEDs(a, ledPins); % Flash all LEDs to indicate game start
    	gameStarted = true;
    	pause(1.5); % Pause before starting
	end
 
	% Show the sequence to the player
	for i = 1:(roundCounter + 1)
    	flashLED(a, ledPins, buttonSequence(i));
    	pause(0.5); % Small delay between flashes
	end
 
	% Check the player's response
	for i = 1:(roundCounter + 1)
    	pressedButton = waitForButtonPress(a, buttonPins, timeLimit);
    	if pressedButton == 0 || pressedButton ~= buttonSequence(i)
        	flashAllLEDs(a, ledPins); % Player failed
        	gameStarted = false;
        	break;
    	end
    	flashLED(a, ledPins, pressedButton); % Confirm correct press
	end
 
	% If the player succeeded in the round
	if gameStarted
    	roundCounter = roundCounter + 1;
    	if roundCounter >= roundsToWin
            flashAllLEDs(a, ledPins); % Player won the game
            gameStarted = false;
    	end
    	pause(0.5); % Pause between rounds
	end
end
 
% Helper Functions
function flashLED(a, ledPins, ledIndex)
    writeDigitalPin(a, ledPins(ledIndex), 1); % Turn LED on
    pause(0.3);                              % Keep it on for 300ms
    writeDigitalPin(a, ledPins(ledIndex), 0); % Turn LED off
end
 
function flashAllLEDs(a, ledPins)
    for i = 1:4
    	writeDigitalPin(a, ledPins(i), 1); % Turn all LEDs on
e    end
    pause(1); % Pause for 1 second
    for i = 1:4
    	writeDigitalPin(a, ledPins(i), 0); % Turn all LEDs off
    end
end
 
function pressedButton = waitForButtonPress(a, buttonPins, timeLimit)
    startTime = tic;
    pressedButton = 0; % Default: no button pressed
    while toc(startTime) < timeLimit
    	for i = 1:4
        	if readDigitalPin(a, buttonPins(i)) == 0
            	pressedButton = i; % Return the button index (1 to 4)
            	return;
        	end
    	end
    end
End
