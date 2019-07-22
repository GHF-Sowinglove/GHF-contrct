![blizzardgame](ghf.png)
# GHF-contrct

Rule analysis
===
* Three generations 25% share rewards 12% - 8% - 5%.


* 15% lottery pool


* 12% Global Weighted Dividend


* 8% contribution


* 1%-5% Miner Award


* 42% enter static pool and insurance pool

Source code analysis
===
Floating income calculation
---
```Solidity
function rand(uint256 _length, address _pAddress) 
        private
        view
        returns(uint256) 
{
	uint256 random = uint256(keccak256(abi.encodePacked(block.difficulty, now, _pAddress, inc_)));
	return random%_length;
}

function inc()
	private
        view
        returns(uint256) 
{
	return rand(100, _pAddress) < 50 ? inc_+=3 : inc+=5;
}

```


## Static income calculation
```Solidity
function calc(address _pAddress, uint256 _canIncome, uint256 _day, uint256 _probFixResW)
        private
        view
        returns(uint256)
{
        uint256 _teth = plyr_[_pAddress].curjoin;
        uint256 _rndrate;
        if(_probFixResW != 0)
            _rndrate = _probFixResW;
        else 
            _rndrate = getFixRetWRand(_pAddress);
        uint256 _income = _teth.mul(_rndrate).mul(_day).div(10000);
        return _canIncome < _income ? _canIncome : _income;
}
```


## Win the prize in the small prize pool
```Solidity
function getWinRecord(address _addr)
        view
        public
        returns(uint256[],uint256[],uint256[],uint256[],uint256[],uint256[])
{
	uint256[] memory r1 = new uint256[](2);
	uint256[] memory r2 = new uint256[](2);
	uint256[] memory r3 = new uint256[](2);
	uint256[] memory r241 = new uint256[](2);
	uint256[] memory r242 = new uint256[](2);
	uint256[] memory r243 = new uint256[](2);
	if(rID_ > 1){
		r1[0] = rID_ - 1;
		r1[1] = plyrRnds_[_addr][r1[0]].win;
	}
	if(rID_ > 2){
		r2[0] = rID_ - 2;
		r2[1] = plyrRnds_[_addr][r2[0]].win;
	}
	if(rID_ > 3){
		
		r3[0] = rID_ - 3;
		r3[1] = plyrRnds_[_addr][r3[0]].win;
	}
	if(r24ID_ > 1){
		r241[0] = r24ID_ - 1;
		r241[1] = plyrRnds24_[_addr][r241[0]].win;
	}

	if(r24ID_ > 2){
		
		r242[0] = r24ID_ - 2;
		r242[1] = plyrRnds24_[_addr][r242[0]].win;
	}
	if(r24ID_ > 3){
		
		r243[0] = r24ID_ - 3;
		r243[1] = plyrRnds24_[_addr][r243[0]].win;
	}
	return(r1, r2, r3, r241, r242, r243);
}
```
