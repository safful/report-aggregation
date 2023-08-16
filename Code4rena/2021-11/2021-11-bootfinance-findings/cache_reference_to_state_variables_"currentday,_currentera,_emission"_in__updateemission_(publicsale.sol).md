## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache Reference To State Variables "currentDay, currentEra, emission" in _updateEmission (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/117) 

# Handle

ye0lde


# Vulnerability details

## Impact

Cache Reference To State Variables "currentDay, currentEra, emission" in _updateEmission (PublicSale.sol)

Caching the references to "currentDay, currentEra, emission" will decrease gas usage. 

## Proof of Concept

The variables "currentDay, currentEra, emission" are referenced 20 times in function "_updateEmission" here:
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L247-L276

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps

I suggest making the changes below to cache those variables:
  
<code>
	function _updateEmission() private {
        uint _now = block.timestamp;                                                                                      // Find now()
        if (_now >= nextDayTime) {                                                                                         // If time passed the next Day time
            
	   uint256 _currentDay = currentDay;
            uint256 _currentEra = currentEra;
            uint256 _emission = emission;

            if (remainingSupply > _emission) {
                remainingSupply -= _emission;                                                 
            }
            else {
                remainingSupply = 0;                                                        
            }
            if (_currentDay >= daysPerEra) {                                                                                 // If time passed the next Era time
                _currentEra += 1; _currentDay = 0;                                                                        // Increment Era, reset Day
                nextEraTime = _now + (secondsPerDay * daysPerEra);                                          // Set next Era time
                _emission = getNextEraEmission();                                                                        // Get correct emission
                mapEra_Emission[currentEra] = _emission;                                                           // Map emission to Era
                emit NewEra(_currentEra, _emission, nextEraTime, totalBurnt);                            // Emit Event
            }
            _currentDay += 1;                                                                                                     // Increment Day
            nextDayTime = _now + secondsPerDay;                                                                  // Set next Day time
            _emission = getDayEmission();                                                                                // Check daily Dmission
            mapEraDay_EmissionRemaining[_currentEra][_currentDay] = _emission;               // Map emission to Day
            uint _era = _currentEra;
            uint _day = _currentDay - 1;
            if (_currentDay == 1) {
                // new era
                _era = _currentEra - 1;
                _day = daysPerEra;
            }

            currentDay = _currentDay;
            currentEra = _currentEra;
            emission = _emission;
			
            emit NewDay(_currentEra, _currentDay, nextDayTime, mapEraDay_Units[_era][_day], mapEraDay_MemberCount[_era][_day]);
        }
    }
</code>


