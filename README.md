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
Level 50 rebate
---
```Solidity
function updateVIPAmount(address _addr, bool isBuy,uint256 _value) private {
        //Update Up to Level 50 Players'Consumption Amount
        uint _counter = 1;
        address _parent = players[_addr].parent;
        address _child = _addr;

        //Three-level rebate
        while(_counter<=50 && _parent!=address(0)){

            Player storage _c = players[_child];
            Player storage _p = players[_parent];

            if(_p.maxChild==address(0)){
                _p.maxChild=_child;
                _p.maxNodeInvest=_c.totalNodeInvest+_c.myInvest;
            }else{
                if(_p.maxNodeInvest<(_c.totalNodeInvest+_c.myInvest)){
                    _p.maxChild=_child;
                    _p.maxNodeInvest=_c.totalNodeInvest+_c.myInvest;
                }
            }

            if(isBuy){
                if(_counter<50){
                    _p.totalNodeInvest+=_value;
                }else{
                    _p.lastTotalNodeInvest+=_value;
                }

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

                uint _minerlvl = minerAddressList.data[_parent].value;
                if(_minerlvl==1){
                    uint256 _bonus = _value*MINER1PERCENT/100;
                    _p.miner1Amount+=_bonus;
                    stats[_parent].allVIP1+=_bonus;
                }else if(_minerlvl==2){
                    uint256 _bonus = _value*MINER2PERCENT/100;
                    _p.miner2Amount+=_bonus;
                    stats[_parent].allVIP2+=_bonus;
                }
            }

            //Update player's VIP level；
            _updateVIP(_parent);
            _child = _parent;
            _parent = players[_parent].parent;
            _counter+=1;
        }
    }

```


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
