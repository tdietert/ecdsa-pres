% Elliptic-curve Cryptography and ECDSA
% Haskell Exchange 2017
% Thomas Dietert

# Elliptic-curve Cryptography

An approach to public-key cryptography based on the algebraic structure of
elliptic curves over finite fields

* Smaller keys (fewer bytes) compared to non-ECC public key cryptography

* Domain is over Galois fields (Finite fields)

* ... but the maths involved are more complex 

# ECDSA

ECDSA, or Elliptic Curve Digital Signature Algorithm allows data to be *signed*
such that the origin of the data can be verified by anyone in possession of the
public key (w/ respect to the signer's private key).

**Nonrepudiation**

... with maths!

# ECDSA Example

An example using the `cryptonite` library in Haskell:

```haskell
ghci> import Crypto.Number.Hash (SHA3_256)
ghci> import Crypto.PubKey.ECC.ECDSA (sign, verify)
ghci> import Crypto.PubKey.ECC.Generate (generate)
ghci> import Crypto.PubKey.ECC.Types (getCurveByName, SEC_p256k1)
```

```haskell
ghci> let msg = "haskellx 2017" :: ByteString
ghci> let secp256k1 = getCurveByName SEC_p256k1
ghci> (pubKey, privKey) <- generate secp256k1
ghci> sig <- sign privKey SHA3_256 msg
ghci> verify SHA3_256 pubKey sig msg
True a
```
# Elliptic Curves

A general form of an **elliptic curve** is the equation 

$$
y^{2} = x^{3} + ax + b \\
4a^{3} + 27b^{2} \neq 0
$$

This is know as _Weierstrass normal form_ (ECs can be defined with different equations)

<img src="img/ecc-chart.svg" alt="Drawing" class="center-img" style="width: 600px"> 

# Group laws

In mathematics, a *group* is an algebraic structure consisting of a set of
elements over which an operation $\oplus$ that combines any two elements of the set
to form a third element within the set. 

The following *group axioms* must also hold:

**Closure**

: $\forall a, b \in G. a \oplus b \in G$

**Associativity**

: $\forall a, b, c \in G. (a \oplus b) \oplus c = a \oplus (b \oplus c)$

**Identity**

: $\exists e, \forall a \in G. e \oplus a = a \oplus e = a$

**Inverse**

: $\forall a, \exists b \in G. a \oplus b = b \oplus a = e$

# Elliptic Curve - Group

Let $O$ be the _point at infinity_, the **identity** element of the group.

> **Note:** _The reason why this is called the "point at infinity" is beyond the scope
of this talk (common in abstract geometry)._

The **inverse** of a point $P = (x,y)$ is $P^{-1} = (x,-y)$ (the point is
reflected over the y-axis, and can also be written $-P$)

The **$\oplus$** **operation** is defined by, given three points $P, Q,
R$ on the elliptic curve, $P + Q + R = O$. Assume this operation to be
associative _and_ commutative.

\ 

Given these properties shown, using this algebraic group structure we can show

$$
P + Q + R^{-1} = O \\
P + Q = R
$$

<img src="img/ecc.png" alt="Drawing" class="center-img" style="width: 900px"> 

# Elliptic Curves over Finite Fields

An **Elliptic Curve** over a finite field $\mathbb{F}_p$ is written as $E(\mathbb{F}_p)$.

These curves are defined by a 6 tuple:
$$(p,a,b,G,n,h)$$ 

where all points on the curve are in the domain of $E(\mathbb{F}_p)$:
$$
\forall x,y\in(\mathbb{F}_p)^{2} \\
\\
y^{2} = x^{3} + ax + b (\mod p) \\
4a^{3} + 27b^{2} (\mod p) \neq 0
$$ 

```
where
  p = large prime 
  a = curve polynomial coefficient
  b = curve polynomial coefficient
  G = generator base point *

  -- Out of the scope of this talk
  n = order (number of points in the group)                        
  h = cofactor      
```

# Elliptic Curves over Finite Fields (cont)

But wait, they actually look like this!

<img src="img/ecc-ff.png" alt="Drawing" class="center-img" style="width: 650px"> 

# Elliptic Curve Public/Private Keys

\ 

Points in $E(\mathbb{F}_p)$ can be multiplied by a scalar $k \in \mathbb{F}_p$

A *Private Key* in ECC is defined by a random scalar $k \in \mathbb{F}_p$

A *Public Key* in ECC is defined by a point $Q = kG$ where $G$ is the generator
point.

\ 

**ECDLP**

: The *Discrete Log Problem* for ECC states that it is computationally infeasable
to find $k$ given $Q = kG$ in $E(\mathbb{F}_p)$

: This is how private keys stay private!

# ECDSA

Allows a party to irrefutably declare that they are the origin of a piece of
data to anyone in the world who posseses their respective *public key*.

**Sign**

: To *sign* a piece of data is to combine an ECC private key and the hashed data
to produce a pair of integers (from which the private key cannot be derived). 

**Verify**

: To *verify* a signed piece of data is to mathematically prove, using the
*public key* of the signer, that the signature was produced using the data and
private key.

# ECDSA - Signature Algorithm

Inputs for the signing function are:

- The data as a string of bytes, $msg$.
- The elliptic curve private-key, $k$.

The **signing** algorithm will be described in terms of the curve Secp256k1, where
$p$ is the secp256k1 prime, and $G$ is it’s respective generator point:

1. Compute $z = Hash(msg)$ (produces an integer, where $z \in \mathbb{F}_p$)
2. \* Generate a random value $d \in \{1,..,p−1\}$
3. Compute $(x,y) = kG$
4. Compute $r = x \mod p$, if $r = 0$, go back to step 1
5. Compute $s = (z + rk) / d$ , if $s = 0$, go back to step 1

\* $d$ must be different for every piece of data signed

# ECDSA - Verify 

Inputs to the verification algorithm are:

- The data as a string of bytes, $msg$
- The signature $(r,s)$
- The EC public key $Q$

The output of the verification algorithm is a boolean indicating whether or not
the data was signed with the *private key* corresponding to the *public key*:

1. Compute $z = Hash(msg)$ (produces an integer, where $z \in \mathbb{F}_p$)
2. Compute $t = (z \mod p) / s$
3. Compute $u = (r \mod p) / s$
4. Let $(x,y) = tG + uQ$
5. Verify that $r = x \mod p$

# Example - cryptonite

Luckily, the haskell `cryptonite` library implements a lot of this machinery for
us!

```haskell
-- | Represent a ECDSA signature namely R and S.
data Signature = Signature
  { sign_r :: Integer -- ^ ECDSA r
  , sign_s :: Integer -- ^ ECDSA s
  } 

-- | ECDSA Private Key.
data PrivateKey = PrivateKey
  { private_curve :: Curve         
  , private_d     :: PrivateNumber -- ^ A random integer
  } 

-- | ECDSA Public Key.
data PublicKey = PublicKey
  { public_curve :: Curve
  , public_q     :: PublicPoint    -- ^ The point derived from the private key 
  } 

sign :: (ByteArrayAccess msg, HashAlgorithm hash, MonadRandom m)
     => PrivateKey -> hash -> msg -> m Signature

verify :: (ByteArrayAccess msg, HashAlgorithm hash) 
       => hash -> PublicKey -> Signature -> msg -> Bool
```

# Example 

```haskell

module Key where

import Crypto.PubKey.ECC.Generate (generate)
import Crypto.PubKey.ECC.Types    (Curve, getCurveByName, SEC_p256k1) 
import Crypto.Hash                (SHA3_256) 

import qualified Crypto.PubKey.ECC.ECDSA as ECDSA 

secp256k1 :: Curve
secp256k1 = getCurveByName SEC_p256k1

genKeyPair :: IO (ECDSA.PublicKey, ECDSA.PrivateKey)
getKeyPair = generate secp256k1

sign :: ECDSA.PrivateKey -> ByteString -> IO ECDSA.Signature
sign pk = sign pk SHA3_256 

verify :: ECDSA.PublicKey -> ECDSA.Signature -> ByteString -> Bool
verify = ECDSA.verify SHA3_256 

```

# Example (cont)

```haskell

module Block where

import Crypto.PubKey.ECC.ECDSA (PrivateKey, PublicKey) 
import Data.Serialize          (Serialize, encode)
import GHC.Generics            (Generic)

import qualified Key

data Signature = Signature 
  { r :: Integer
  , s :: Integer
  } deriving (Generic, Serialize)

data BlockHeader = BlockHeader
  { origin       :: PublicKey    -- ^ Public key of signer
  , previousHash :: ByteString   -- ^ Hash of previous block
  , blockData    :: [ByteString] -- ^ Data in Block
  } deriving (Generic, Serialize)

data Block = Block
  { index     :: Int          -- ^ Block index
  , header    :: BlockHeader  -- ^ Header (piece of data that is signed)
  , signature :: Signature    -- ^ Signature of header
  } deriving (Generic, Serialize)
```

# Example (cont)

``` haskell
module Block where

...

makeBlock 
  :: (Serialize a)
  => (PrivateKey, PublicKey) -- Signer's KeyPair
  -> Block                   -- Previous Block
  -> [a]                     -- Block Data
  -> IO Block                
makeBlock (pubKey, privKey) prevBlock blockData = do
   
    -- * Where the magic happens
    blockSig <- sign privKey $ S.encode blockHeader 
    
    pure $ Block 
      { index     = index prevBlock + 1
      , header    = blockHeader
      , signature = blockSig
      }
  where
    blockHeader = BlockHeader 
      { origin       = pubKey
      , previousHash = hashBlock prevBlock
      , blockData    = map encode blockData 
      }

```

# Example (cont)

``` haskell

module Block where 

...

verifyBlock :: Block -> Bool
verifyBlock block = 
    Key.verify (origin header) (signature block) (encode header) 
  where
    blkHeader = header block

```

And finally...

```haskell
ghci> import Block
ghci> import Key
ghci> keys <- genKeyPair
ghci> let blockData = [1,2,3,4,5]
ghci> block <- makeBlock keys genesisBlock blockData  
ghci> verifyBlock block
True
``` 
