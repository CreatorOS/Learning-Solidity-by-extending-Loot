# Learning Solidity by extending Loot!
We do assume you know the basics of Ethereum and solidity for this quest. We’re going to dive into a really exciting project called [Loot](https://www.lootproject.com/) that was announced in the first week of September, and another called mloot as an extension. You’d need to have done the basics of ethereum quests from [https://questb.uk](https://questb.uk) and the quest on NFTs to follow this quest and build on top.team, you own that nft. To increase your chances of winning you can attach bandit gold coins before entering the fight. If you lose, the coins go into a lootbox. If you win, you earn all the coins in the lootbox. 

We’ll be building a game. If you’re familiar with loot, you’d know that Loot is an NFT which describes a unique character from a mystical fictional land. What we’ll do is let people use their loot characters to fight with bandits. Bandits are also loot like characters. We’ll call this game banditloot. 

Here are the rules of the game. 

You must defeat a bandit to win. To defeat a Bandit you must call a fight() function with the loot token you want to play. A random Bandit is created in every block. Your Loot token character will be pitted against the bandit and a fight algorithm will be triggered. When you defeat a bandit, that bandit becomes a part of your collection/wallet.

So, we’ll be - by virtue of launching this game - creating a new crypto currency and a new NFT collection of the characters we introduce in our smart contracts - called BanditLoots.

We will be using some advance techniques that will come very handy in your solidity journey.

Throughout this quest we’ll be using the following codebase and walking through it step by step.

[https://remix.ethereum.org/\#version=soljson-v0.8.7\+commit.e28d00a7.js&optimize=false&runs=200&gist=e64f955848f373298e45736b4ca29420](https://remix.ethereum.org/#version=soljson-v0.8.7+commit.e28d00a7.js&optimize=false&runs=200&gist=e64f955848f373298e45736b4ca29420)

At the end of this quest there is a bounty worth 1ETH!
## Let’s launch a bandit gold coin ERC20 token
This time around, instead of writing the entire token’s contract ourselves, we’ll cheat a little bit. ERC20 is so common that folks have already created libraries. These libraries are not only easier to use but they’re also secure and optimized. The most popular and vetted repository is that of [openzepplin](https://openzeppelin.com/contracts/).

We’ll import open zepplin’s open library using 	

```

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract BanditLootGold is ERC20 {

    constructor(address creator) ERC20("BanditLootGold","BGLD"){

    }

}

```

We’re extending the ERC20 contract defined by openzepplin and saying our BanditLootGold coin will extend the same, using the `is` keyword.

Every cryptocurrency has a name and a symbol. The ERC20 contract on openzepplin expects these two to be parameters to the constructor. 

```

    constructor(address creator) ERC20("BanditLootGold","BGLD"){

    }

```

Whenever the constructor of BanditLootGold coin is called, the ERC20 constructor is also called with the name and symbol as parameters. “BanditLootGold” and “BGLD” respectively.
## Making sure there are users for BanditGold
There are hundreds of erc20 tokens launching every single day. But most of them die silently - because no one uses them, nobody buys them, nobody sells them.

Now that we’ve created our token, let's distribute it smartly.

This is a technique called Airdropping. You give the tokens to some people for free so that they start using the coin and trade starts to happen - thereby putting into motion a flywheel for more people and start trading the coin. 

In BanditLoot, we’ll distribute 1000 tokens to everyone who holds a loot NFT.

First, we’ll define the interface of the function that we want to use from the loot contract ([https://etherscan.io/address/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7\#code](https://etherscan.io/address/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7#code))

```

interface Loot { 

    function ownerOf(uint tokenId) external returns(address);

}

```

We’ll also initialize the loot object. We do so by providing the address of the Loot Contract. 

```

    Loot loot = Loot(0xFF9C1b15B16263C61d017ee9F65C50e4AE0113D7);

    Loot mloot = Loot(0x1dfe7Ca09e99d10835Bf73044a23B73Fc20623DF);

```

Loot and mLoot are both projects that expose the same interfaces. So, we’ve conveniently used the same `interface Loot` for both of them. 

We’ll create a function where anyone who owns a Loot (or an mLoot) will get some free coins. 

We’ll call this function `claimAirdrop`

```

    mapping(uint => bool) airdropClaimMap;

    function claimAirdrop(uint lootNumber) public {

        if((loot.ownerOf(lootNumber) == msg.sender) && (!airdropClaimMap\[lootNumber\])){

            _mint(msg.sender, 1000 \* 1e18);

            airdropClaimMap\[lootNumber\] = true;

        }

    }

```

The _mint is a function in the ERC20 contract written by open zeppelin. We’ll be minting 1000 coins to anyone who owns a particular loot, by checking if the said person is the owner of the loot in the first place. The 1e18 is because the lowest denomination on this coin is going to be 10^-18. Similar to Eth and wei we had seen earlier. We have to calculate all the transactions in the lowest denomination because Solidity doesn’t support decimals (yet).
## Creating a Bandit NFT
Similar to using ERC20 token from open zeppelin, we’ll use the ERC721 token also from the same repo. We’ll call this NFT BanditLoot. Or b-loot.

```

contract BanditLoot is ERC721URIStorage {

    constructor() ERC721("BanditLoot", "B-LOOT") {

    }

}

```

ERC721URIStorage gives us all the functionality of an NFT but also exposes an additional function `tokenURI`. This allows us to generate an image for making it user friendly. Check this image out on this NFT [https://opensea.io/assets/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7/1](https://opensea.io/assets/0xff9c1b15b16263c61d017ee9f65c50e4ae0113d7/1)

It’s a simple text based image. But this image was generated completely using Solidity. The way to do that is to populate the tokenURI so as to return an SVG. 

```

    function tokenURI(uint256 tokenId) override public view returns (string memory) {

        string\[5\] memory parts;

        parts\[0\] = '.base { fill: white; font-family: serif; font-size: 14px; }';

        parts\[1\] = ‘NFT\#’;

        parts\[2\] = '';

        parts\[3\] = toString(tokenId);

        parts\[4\] = '';

        string memory output = string(abi.encodePacked(parts\[0\], parts\[1\], parts\[2\], parts\[3\], parts\[4\]));

        

        string memory json = Base64.encode(bytes(string(abi.encodePacked('{"name": "Bag \#', toString(tokenId), '", "description": "Bandit Loot is a game based on randomized odds and cryptographical proofs and verifiablility of victory.", "image": "data:image/svg\+xml;base64,', Base64.encode(bytes(output)), '"}'))));

        output = string(abi.encodePacked('data:application/json;base64,', json));

        return output;

    }

```

To do string concatenations [we must use `string(abi.encodePacked())`](https://docs.soliditylang.org/en/latest/units-and-global-variables.html?highlight=abi.encode#abi-encoding-and-decoding-functions)

The function itself must return a JSON that is Base64 encoded. The JSON must include the name of the NFT, a description and the image itself in SVG. 

If these are present the UI apps know what to display against this NFT.
## Copying Loot contract and building on top
Our bandit loot will work with the similar concept as that of loot. Such that the armour names are the same. This will allow Bandit Loot to be composable. Composability means that other developers can build on top of it, the same way we’re building on top of loot. So keeping the interfaces the same as Loot will make it easier for people to build on top. Maybe someone will create a contract using loot, mLoot and banditLoot!

So let’s just go and copy the entire contract from mLoot and paste it in our contract. We’ll now make modifications to this contract. 

The only way to earn a BanditLoot NFT is to win a fight with that Bandit. However instead of being able to just claim it, we want to enable a fight. 

So we’ll remove the claim() function and include a fight() function.
## Fighting logic
There are so many ways to fight. But we’ll use one such algorithm to fight. 

To fight, the player must call a function fight() and pass a tokenId. This NFT (loot or mLoot) must be owned by the player. If the ownership is verified, the fight starts. 

Here are some more dynamics of the game designed here:

The bandit is selected randomly. There is one bandit available to fight in one block. One bandit can be defeated only once. The first person to beat that bandit becomes the winner of that NFT.

The winner is selected depending on another random parameter called accuracy. Accuracy can range from 0 to 1. To improve the accuracy of the attack, the player can send bandit gold - the erc20 token we had created earlier. 

Here’s the core element of the fighting logic :

```

            playerScore \+= 10 \+ (((banditArmors\[i\] \+ playerWeapon)%arrayLengths\[i\]) \* accuracy \* 10)/1000000 - greatness(banditArmors\[i\]);

```

Depending on the armor of the bandit and the weapon, a score is calculated. 

If the player wins the fight, they earn the bandit loot NFT and some bandit loot gold, defined by the fight scores. Explore the contract’s fight() function to see how this is implemented. Feel free to make modifications!
## Securing our Bandit Gold
Whenever a player beats a bandit we want to send them some bandit gold. 

This bandit gold can be created only by our bandit loot contract. Noone should be able to mint coins  for themselves. So we’ll have to create a couple of checks. 

First up, instead of deploying the bandit gold (erc20) contract separately, we’ll deploy only the banditloot erc721 contract and have that contract deploy the bandit gold contract. Let’s ask our BanditLoot constructor to deploy the BanditLootGold contract whenever the constructor is called. 

```

    BanditLootGold banditLootGold;

    

    constructor() ERC721("BanditLoot", "B-LOOT") {

        banditLootGold = new BanditLootGold();

    }

```

BanditLootGold is the contract that has now been deployed by the BanditLoot contract. 

Let’s modify our BanditLootGold erc20 contract to store who deployed this contract. 

```    
    constructor() ERC20("BanditLootGold","BGLD"){

        owner = msg.sender;

    }

```

This stores the owner in a variable based on who called the constructor. So the owner of this contract is the other contract that deployed it.

Now let’s use this information. 

Once the fight concludes, the bandit gold needs to be minted and into the wallet of the user. So first let’s create that function : 

```

    function mint(address to, uint value) external {

        _mint(to, value);

    }

```

We need to wrap the _mint() function in the mint() function because the _mint() function exposed by ERC20 openzeppelin contract sets the visibility of the function as `internal` meaning it cannot be called from another contract. So, we’ll wrap it in our mint function that has a visibility of `external`.

But OpenZeppelin had made it internal for a reason. You don’t want anyone to be calling this function and minting coins for themselves.

So in our mint function, we’ll say that this function can be called only by the owner of this contract. 

```

    function mint(address to, uint value) external {

        require(msg.sender == owner, "Only BanditLoot Contract can mint");

        _mint(to, value);

    }

```

Adding a check at the beginning of the function to check who is sending this request to mint. If the sender is the contract BanditLoot, we’ll allow the minting to happen, else fail the minting.
## Bandits don’t own jewels
Now that you’d have the entire contract in place, we’ll make some modifications from the loot project. Loot has necklaces, rings that bandits obviously don’t own. So we’ll remove these attributes from our contract when copying from mLoot. But, we’ll also cheekily remove one more attribute `footArmor`. Not because Bandits don’t have foot armor, but because we’ll otherwise run into a very interesting problem on Solidity. 

Solidity allows only Contracts of total length less than 24KB in size. We remove necklaces, rings and foot armor to make our contract shorter.
## Make a better fight & earn 1 ETH
Now check out the entire contract and see if you can design a better fighting algorithm. 

Can you also try to make the banditloot contract look cooler?

You can win 1ETH by suggesting the best update to this contract : [https://github.com/madhavanmalolan/banditloot](https://github.com/madhavanmalolan/banditloot)