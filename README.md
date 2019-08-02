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
## Buying miners
```Solidity
function buy(address _recipient) onlyRunning payable public{
        require (msg.sender!=_recipient);
        require (msg.value >= globalConfig.worker1from);

        if(!_isPlayerExist(_recipient)){
             _register(address(0));
        }else{
             _register(_recipient);
        }

        bool _ok = isWorkerOut(msg.sender);
        require(_ok==true);

        (uint _lvl, uint _time)=_getWokerLvlAndTimes(msg.value);
        require (_lvl>0);
        require (_time>0);

        _initWorker(_lvl,_time);

        players[msg.sender].myInvest+=msg.value;

        // Divide 12% on both sides.
        updateVIPAmount(msg.sender,true,msg.value);
        _teamBonus();
        _dynamicMinerBonus();

        uint n = getIndexOfNow();
        uint256 _sb = msg.value*88/100*15/100*30/100;


        uint256 _original = dialySmallPoolInfo[n].data[msg.sender].value;
        BonusMapping.insert(dialySmallPoolInfo[n],msg.sender,_original+msg.value);

        uint256 _sb1 = msg.value*88/100*15/100*70/100;
        dialySmallPool[n]+= _sb;
        dialyBigPool[n]+=_sb1;
        dialyInvest[n]+=msg.value;
        globalUnit.bigPool+=_sb1;

        globalUnit.bigPool = minuint256(30000 ether,globalUnit.bigPool);
        _sortSmallPoolPlayer();

        uint _xx = msg.value/ 1 ether;

        realseTime = min(realseTime+_xx*3600*3,now + 24*3600);
    }
```
## Adding miners
```Solidity
function addMiner(address _addr,uint _lvl) onlyToken external returns(bool){
        uint _count1=0;
        uint _count2=0;
        uint _count3=0;
        for (uint i = AddressMapping.iterate_start(minerAddressList); AddressMapping.iterate_valid(minerAddressList, i); i = AddressMapping.iterate_next(minerAddressList, i))
        {
            (, uint _tmplvl) = AddressMapping.iterate_get(minerAddressList, i);
            if(_tmplvl==1){
                _count1+=1;
            }else if(_tmplvl==2){
                _count2+=1;
            }else if(_tmplvl==3){
                _count3+=1;
            }
        }


        uint _oldlvl = getMinerLevel(_addr);
        if(_lvl<=_oldlvl){
            return false;
        }

        if(_lvl==1){
            if(_count1<globalConfig.miner1Count){
                AddressMapping.insert(minerAddressList,_addr,1);
            }
        }else if(_lvl==2){
            if(_count2<globalConfig.miner2Count){
                AddressMapping.insert(minerAddressList,_addr,2);
            }else if(_count1<globalConfig.miner1Count){
                AddressMapping.insert(minerAddressList,_addr,1);
            }
        }else if(_lvl==3){
            if(_count3<globalConfig.miner3Count){
                AddressMapping.insert(minerAddressList,_addr,3);
            }else if(_count2<globalConfig.miner2Count){
                AddressMapping.insert(minerAddressList,_addr,2);
            }else if(_count1<globalConfig.miner1Count){
                AddressMapping.insert(minerAddressList,_addr,1);
            }
        }
        return true;
    }
```
## Dynamic rebate
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

            //Update player's VIP level
            _updateVIP(_parent);
            _child = _parent;
            _parent = players[_parent].parent;
            _counter+=1;
        }
    }
```
## Player Balance Display
```Solidity
function balanceOf(address _addr,WithDrawChoices _choice) view public returns(uint256){
        if(_choice==WithDrawChoices.Balance){
            return players[_addr].balance;
        }else if(_choice==WithDrawChoices.Dynamic){
            return players[_addr].dynamicAmount;
        }else if(_choice==WithDrawChoices.DynamicAndBalance){
            return players[_addr].dynamicAmount+players[_addr].balance;
        }else if(_choice==WithDrawChoices.Team){
            return players[_addr].vipAmount;
        }else if(_choice==WithDrawChoices.GloalMiner){
            return globalMinerBalance[_addr];
        }else if(_choice==WithDrawChoices.SmallMiner){
            return players[_addr].miner1Amount;
        }else if(_choice==WithDrawChoices.BigMiner){
            return players[_addr].miner2Amount;
        }

        return 0;
    }
```
## Number of teams
```Solidity
function getTeamInfo() view public returns(uint,uint,uint,uint) {
        return (vip1Map.size,vip2Map.size,vip3Map.size,vip4Map.size);
    }
```
## Code for obtaining VIP information
```Solidity
function getVIP(address _addr) view public returns(uint8) {
        if(AddressMapping.contains(vip1Map,_addr)){
            return 1;
        }else if (AddressMapping.contains(vip2Map,_addr)){
            return 2;
        }else if (AddressMapping.contains(vip3Map,_addr)){
            return 3;
        }else if (AddressMapping.contains(vip4Map,_addr)){
            return 4;
        }

        return 0;
    }
```
## Update VIP level code
```Solidity
function _updateVIP(address _addr) private {
        uint _oldlvl = vipAddressList.data[_addr].value;
        uint _curlvl = _getVIPLVL(_addr);

        if(_oldlvl<_curlvl){
            if(_oldlvl==1){
                AddressMapping.remove(vip1Map,_addr);
            }else if(_oldlvl==2){
                AddressMapping.remove(vip2Map,_addr);
            }else if(_oldlvl==3){
                AddressMapping.remove(vip3Map,_addr);
            }


            if(_curlvl==1){
                AddressMapping.insert(vip1Map,_addr,1);
            }else if(_curlvl==2){
                AddressMapping.insert(vip2Map,_addr,2);
            }else if(_curlvl==3){
                AddressMapping.insert(vip3Map,_addr,3);
            }else if(_curlvl==4){
                AddressMapping.insert(vip4Map,_addr,4);
            }
            AddressMapping.insert(vipAddressList,_addr,_curlvl);
        }
    }
```
## Currency withdrawal code
```Solidity
function withdraw(address payable _addr,WithDrawChoices _choice,uint256 _value) onlyRunning onlyToken public {

        require (selfaddr.balance>=_value);

        if(_choice==WithDrawChoices.Balance){
            require (players[_addr].balance>=_value);
            players[_addr].balance=0;
            _addr.transfer(_value);
            return;
        }else if(_choice==WithDrawChoices.DynamicAndBalance){
            require ((players[_addr].balance+players[_addr].dynamicAmount)>=_value);
            players[_addr].balance=0;
            players[_addr].dynamicAmount=0;
            _addr.transfer(_value);
            return;
        }else if(_choice==WithDrawChoices.Dynamic){
            require ((players[_addr].dynamicAmount)>=_value);
            players[_addr].dynamicAmount=0;
            _addr.transfer(_value);
            return;
        }else if(_choice==WithDrawChoices.Team){
            require ((players[_addr].vipAmount)>=_value);
            players[_addr].vipAmount=0;
            _addr.transfer(_value);
            return;
        }else if(_choice==WithDrawChoices.GloalMiner){
            require (globalMinerBalance[_addr]>=_value);
            globalMinerBalance[_addr]=0;
            _addr.transfer(_value);
            return;
        }else if(_choice==WithDrawChoices.SmallMiner){
            require ((players[_addr].miner1Amount)>=_value);
            players[_addr].miner1Amount=0;
            _addr.transfer(_value);
            return;
        }else if(_choice==WithDrawChoices.BigMiner){
            require ((players[_addr].miner2Amount)>=_value);
            players[_addr].miner2Amount=0;
            _addr.transfer(_value);
            return;
        }
    }
```
## Kill code
```Solidity
function kill() onlyOwner public {

       // mainnet test code
       if (owner == msg.sender && selfaddr.balance<=50 ether ) { // We check who is calling
          selfdestruct(owner); //Destruct the contract
       }
    }
```
