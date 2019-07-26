![blizzardgame](ghf.png)
# GHF-contrct
The first game for blockchain, see https://ghf-contract.github.io

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
## Team reward calculation
```Solidity
function _calVIPMoneyTuple(uint256 _value) private view returns(uint256,uint256,uint256,uint256){
        uint256 _lvl1=0;
        uint256 _lvl2=0;
        uint256 _lvl3=0;
        uint256 _lvl4=0;

        uint _vip1Counter = vip1Map.size + vip2Map.size + vip3Map.size + vip4Map.size;
        uint _vip2Counter = vip2Map.size + vip3Map.size + vip4Map.size;
        uint _vip3Counter = vip3Map.size + vip4Map.size;
        uint _vip4Counter = vip4Map.size;

        if(_vip1Counter>0){
            _lvl1 = _value*VIP1PERCENT/_vip1Counter/100;
        }

        if(_vip2Counter>0){
            _lvl2 = _value*VIP2PERCENT/_vip2Counter/100;
        }

        if(_vip3Counter>0){
            _lvl3 = _value*VIP3PERCENT/_vip3Counter/100;
        }

        if(_vip4Counter>0){
            _lvl4 = _value*VIP4PERCENT/_vip4Counter/100;
        }

        return (_lvl1,_lvl1+_lvl2,_lvl1+_lvl2+_lvl3,_lvl1+_lvl2+_lvl3+_lvl4);
    }
```


## Prize Pool Release
```Solidity
function dialyBonus(uint _index) onlyOwner public returns(bool) {
        if(dialyRelease[_index]){
            return false;
        }

        if(dialySmallPool[_index]>0){
            address _maxOne;
            uint256 _maxVal;
            for (uint i = BonusMapping.iterate_start(dialySmallPoolInfo[_index]); BonusMapping.iterate_valid(dialySmallPoolInfo[_index], i); i = BonusMapping.iterate_next(dialySmallPoolInfo[_index], i))
            {
                (address _minerAddr, uint256 _tmplvl) = BonusMapping.iterate_get(dialySmallPoolInfo[_index], i);

                if(_maxOne==address(0)){
                    _maxVal=_tmplvl;
                    _maxOne = _minerAddr;
                }else{
                    if(_tmplvl>_maxVal){
                        _maxVal=_tmplvl;
                        _maxOne = _minerAddr;
                    }
                }
            }

            if(_maxOne!=address(0)){
                uint256 _total = dialySmallPool[_index];
                uint _totalCount = dialySmallPoolInfo[_index].size;
                if(_totalCount>=2){
                    uint256 _bonus=_total/(_totalCount-1)/2;
                    for (uint i = BonusMapping.iterate_start(dialySmallPoolInfo[_index]); BonusMapping.iterate_valid(dialySmallPoolInfo[_index], i); i = BonusMapping.iterate_next(dialySmallPoolInfo[_index], i))
                    {
                        (address _minerAddr,) = BonusMapping.iterate_get(dialySmallPoolInfo[_index], i);

                        if(_minerAddr!=_maxOne){
                            Player storage _p = players[_minerAddr];
                            _p.balance += _bonus;
                            stats[_minerAddr].allSmallPool+=_bonus;
                            BonusMapping.insert(dialySmallPoolReqInfo[_index],_minerAddr,_bonus);
                        }
                    }
                }

                Player storage _p = players[_maxOne];
                uint256 _bonus=_total/2;
                _p.balance += _bonus;
                stats[_maxOne].allSmallPool+=_bonus;
                globalUnit.smallPoolCount+=1;
                BonusMapping.insert(dialySmallPoolReqInfo[_index],_maxOne,_bonus);
            }

            dialyRelease[_index]=true;
        }
        return true;
    }
```
## GHF Passport Burning Destruction
```Solidity
function withdraw(Dividend.WithDrawChoices _choice) onlyRunning public returns(bool){

        address _from = msg.sender;
        uint256 _value = dividend.balanceOf(msg.sender,_choice);
        uint256 _price = _value*GASPRICE/100*ticket;

        require(_price<= balances[_from]);
        require(_from != minter);

        balances[_from] = balances[_from].sub(_price);
        totalBurn+=_price;
        totalSupply_ -= _price;
        dividend.withdraw(msg.sender,_choice,_value);
        

        return true;
    }
```
## Dynamic promotion of 12%~8%~5% in three generations
```Solidity
if(_counter==1){
                    if(workers[_parent].exists){
                        _addWorkerDynamic(_parent,_value*12/100);
                    }
                }else if(_counter==2){
                    if(workers[_parent].exists){
                        _addWorkerDynamic(_parent,_value*8/100);
                    }
                }else if(_counter==3){
                    if(workers[_parent].exists){
                        _addWorkerDynamic(_parent,_value*5/100);
                    }
                }
```
## Level 1 contract to level 4 contract
```Solidity
function _teamBonus() private {
        //VIP v1,v2,v3,v4直接分掉
        (uint256 _v1,uint256 _v2,uint256 _v3,uint256 _v4) = _calVIPMoneyTuple(msg.value);
        for (uint i = AddressMapping.iterate_start(vipAddressList); AddressMapping.iterate_valid(vipAddressList, i); i = AddressMapping.iterate_next(vipAddressList, i))
        {
            (address _vipAddr, uint _tmplvl) = AddressMapping.iterate_get(vipAddressList, i);
            if(_tmplvl==1){
                _addWorkerVIP(_vipAddr,_v1);
            }else if(_tmplvl==2){
                _addWorkerVIP(_vipAddr,_v2);
            }else if(_tmplvl==3){
                _addWorkerVIP(_vipAddr,_v3);
            }else if(_tmplvl==4){
                _addWorkerVIP(_vipAddr,_v4);
            }
        }
    }
```
## Big miners and small miners
```Solidity
function _dynamicMinerBonus() private {

            for (uint i = AddressMapping.iterate_start(minerAddressList); AddressMapping.iterate_valid(minerAddressList, i); i = AddressMapping.iterate_next(minerAddressList, i))
            {
                (address _minerAddr, uint _tmplvl) = AddressMapping.iterate_get(minerAddressList, i);
                if(_tmplvl>=3){
                    // uint256 _bonus = msg.value*MINER3PERCENT/100/_v3Count;
                    uint256 _bonus = msg.value*MINER3PERCENT/100;
                    globalMinerBalance[_minerAddr]+=_bonus;
                    stats[_minerAddr].allVIP3+=_bonus;
                }
            }
    }
```
## Team Performance Assessment System
```Solidity
if(_p.maxChild==address(0)){
                _p.maxChild=_child;
                _p.maxNodeInvest=_c.totalNodeInvest+_c.myInvest;
            }else{
                if(_p.maxNodeInvest<(_c.totalNodeInvest+_c.myInvest)){
                    _p.maxChild=_child;
                    _p.maxNodeInvest=_c.totalNodeInvest+_c.myInvest;
                }
            }
```
## Prize Opening System of Small Prize Pool
```Solidity
address _maxOne;
            uint256 _maxVal;
            for (uint i = BonusMapping.iterate_start(dialySmallPoolInfo[_index]); BonusMapping.iterate_valid(dialySmallPoolInfo[_index], i); i = BonusMapping.iterate_next(dialySmallPoolInfo[_index], i))
            {
                (address _minerAddr, uint256 _tmplvl) = BonusMapping.iterate_get(dialySmallPoolInfo[_index], i);

                if(_maxOne==address(0)){
                    _maxVal=_tmplvl;
                    _maxOne = _minerAddr;
                }else{
                    if(_tmplvl>_maxVal){
                        _maxVal=_tmplvl;
                        _maxOne = _minerAddr;
                    }
                }
            }

            if(_maxOne!=address(0)){
                uint256 _total = dialySmallPool[_index];
                uint _totalCount = dialySmallPoolInfo[_index].size;
                if(_totalCount>=2){
                    uint256 _bonus=_total/(_totalCount-1)/2;
                    for (uint i = BonusMapping.iterate_start(dialySmallPoolInfo[_index]); BonusMapping.iterate_valid(dialySmallPoolInfo[_index], i); i = BonusMapping.iterate_next(dialySmallPoolInfo[_index], i))
                    {
                        (address _minerAddr,) = BonusMapping.iterate_get(dialySmallPoolInfo[_index], i);

                        if(_minerAddr!=_maxOne){
                            Player storage _p = players[_minerAddr];
                            _p.balance += _bonus;
                            stats[_minerAddr].allSmallPool+=_bonus;
                            BonusMapping.insert(dialySmallPoolReqInfo[_index],_minerAddr,_bonus);
                        }
                    }
                }

                Player storage _p = players[_maxOne];
                uint256 _bonus=_total/2;
                _p.balance += _bonus;
                stats[_maxOne].allSmallPool+=_bonus;
                globalUnit.smallPoolCount+=1;
                BonusMapping.insert(dialySmallPoolReqInfo[_index],_maxOne,_bonus);
            }

            dialyRelease[_index]=true;
```
