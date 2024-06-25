---
layout: post
title: DID 学习日记 - RHS里面的 key/path的细节
subtitle:
categories: DID PolygonID Web3
tags: [DID, PolygonID, Web3, RHS]
---

## DID 学习日记 - RHS 里面的 key/path 的细节

RHS 链接: <https://docs.iden3.io/services/rhs/>

这里面有个逻辑
<https://github.com/iden3/merkletree-proof/blob/63e8bb011e0ef7ff4d881c9835b5e687d4eb4cda/proof.go#L122>

```go
func GenerateProof(ctx context.Context, cli NodeReader,
	treeRoot *merkletree.Hash,
	key *merkletree.Hash) (*merkletree.Proof, error) {

	var exists bool
	var siblings []*merkletree.Hash
	var nodeAux *merkletree.NodeAux

	mkProof := func() (*merkletree.Proof, error) {
		return merkletree.NewProofFromData(exists, siblings, nodeAux)
	}

	nextKey := treeRoot
	for depth := uint(0); depth < uint(len(key)*8); depth++ {
		if *nextKey == merkletree.HashZero {
			return mkProof()
		}
		n, err := cli.GetNode(ctx, nextKey)
		if err != nil {
			return nil, err
		}
		switch nt := n.Type(); nt {
		case NodeTypeLeaf:
			if bytes.Equal(key[:], n.Children[0][:]) {
				exists = true
				return mkProof()
			}
			// We found a leaf whose entry didn't match hIndex
			nodeAux = &merkletree.NodeAux{
				Key:   n.Children[0],
				Value: n.Children[1],
			}
			return mkProof()
		case NodeTypeMiddle:
			if merkletree.TestBit(key[:], depth) {
				nextKey = n.Children[1]
				siblings = append(siblings, n.Children[0])
			} else {
				nextKey = n.Children[0]
				siblings = append(siblings, n.Children[1])
			}
		default:
			return nil, fmt.Errorf(
				"found unexpected node type in tree (%v): %v",
				nt, n.Hash.Hex())
		}
	}

	return nil, errors.New("tree depth is too high")
}
```

通过 key 的第 n 个 bit 是 0 还是 1 来决定找左右子节点

更细节的代码在这里: <https://github.com/iden3/go-merkletree-sql/blob/d140a683025a08f6e6123272fcef89dae8b6f9f3/merkletree.go#L798>

```go
// GenerateProof generates the proof of existence (or non-existence) of an
// Entry's hash Index for a Merkle Tree given the root.
// If the rootKey is nil, the current merkletree root is used
func (mt *MerkleTree) GenerateProof(ctx context.Context, k *big.Int,
	rootKey *Hash) (*Proof, *big.Int, error) {
	p := &Proof{}
	var siblingKey *Hash

	kHash, err := NewHashFromBigInt(k)
	if err != nil {
		return nil, nil, err
	}
	path := getPath(mt.maxLevels, kHash[:])
	if rootKey == nil {
		rootKey = mt.Root()
	}
	nextKey := rootKey
	for p.depth = 0; p.depth < uint(mt.maxLevels); p.depth++ {
		n, err := mt.GetNode(ctx, nextKey)
		if err != nil {
			return nil, nil, err
		}
		switch n.Type {
		case NodeTypeEmpty:
			return p, big.NewInt(0), nil
		case NodeTypeLeaf:
			if bytes.Equal(kHash[:], n.Entry[0][:]) {
				p.Existence = true
				return p, n.Entry[1].BigInt(), nil
			}
			// We found a leaf whose entry didn't match hIndex
			p.NodeAux = &NodeAux{Key: n.Entry[0], Value: n.Entry[1]}
			return p, n.Entry[1].BigInt(), nil
		case NodeTypeMiddle:
			if path[p.depth] {
				nextKey = n.ChildR
				siblingKey = n.ChildL
			} else {
				nextKey = n.ChildL
				siblingKey = n.ChildR
			}
		default:
			return nil, nil, ErrInvalidNodeFound
		}
		if !bytes.Equal(siblingKey[:], HashZero[:]) {
			SetBitBigEndian(p.notempties[:], p.depth)
			p.siblings = append(p.siblings, siblingKey)
		}
	}
	return nil, nil, ErrKeyNotFound
}
```

这里的 `if path[p.depth]` 中的 path 是哪里来的呢?

从<https://github.com/iden3/go-merkletree-sql/blob/d140a683025a08f6e6123272fcef89dae8b6f9f3/merkletree.go#L114C1-L141C2> 这里的代码看

这个 k 或者 path 是自己添加的时候指定的

```go
// Add adds a Key & Value into the MerkleTree. Where the `k` determines the
// path from the Root to the Leaf.
func (mt *MerkleTree) Add(ctx context.Context, k, v *big.Int) error {
	// verify that the MerkleTree is writable
	if !mt.writable {
		return ErrNotWritable
	}

	kHash, err := NewHashFromBigInt(k)
	if err != nil {
		return fmt.Errorf("can't create hash from Key: %w", err)
	}
	vHash, err := NewHashFromBigInt(v)
	if err != nil {
		return fmt.Errorf("can't create hash from Value: %w", err)
	}

	mt.Lock()
	defer mt.Unlock()

	newNodeLeaf := NewNodeLeaf(kHash, vHash)
	path := getPath(mt.maxLevels, kHash[:])

	newRootKey, err := mt.addLeaf(ctx, newNodeLeaf, mt.rootKey, 0, path)
	if err != nil {
		return err
	}
	mt.rootKey = newRootKey
	return mt.db.SetRoot(ctx, mt.rootKey)
}

```

那么回到 issuer 代码, 大概经过下面这几段之后

<https://github.com/0xPolygonID/issuer-node/blob/84bcb60ace21783051ad8787f35f3585dfbdc70e/internal/api_ui/responses.go#L88C1-L133C2>

```go
func credentialResponse(w3c *verifiable.W3CCredential, credential *domain.Claim) Credential {
	var expiresAt *TimeUTC
	expired := false
	if w3c.Expiration != nil {
		if time.Now().UTC().After(w3c.Expiration.UTC()) {
			expired = true
		}
		expiresAt = common.ToPointer(TimeUTC(*w3c.Expiration))
	}

	proofs := getProofs(credential)

	var refreshService *RefreshService
	if w3c.RefreshService != nil {
		refreshService = &RefreshService{
			Id:   w3c.RefreshService.ID,
			Type: RefreshServiceType(w3c.RefreshService.Type),
		}
	}

	var displayService *DisplayMethod
	if w3c.DisplayMethod != nil {
		displayService = &DisplayMethod{
			Id:   w3c.DisplayMethod.ID,
			Type: DisplayMethodType(w3c.DisplayMethod.Type),
		}
	}

	return Credential{
		CredentialSubject:     w3c.CredentialSubject,
		CreatedAt:             TimeUTC(*w3c.IssuanceDate),
		Expired:               expired,
		ExpiresAt:             expiresAt,
		Id:                    credential.ID,
		ProofTypes:            proofs,
		RevNonce:              uint64(credential.RevNonce),
		Revoked:               credential.Revoked,
		SchemaHash:            credential.SchemaHash,
		SchemaType:            shortType(credential.SchemaType),
		SchemaUrl:             credential.SchemaURL,
		UserID:                credential.OtherIdentifier,
		SchemaTypeDescription: credential.SchemaTypeDescription,
		RefreshService:        refreshService,
		DisplayMethod:         displayService,
	}
}

```

<https://github.com/0xPolygonID/issuer-node/blob/84bcb60ace21783051ad8787f35f3585dfbdc70e/internal/core/services/claims.go#L92C1-L113C2>

```golang
/ Save creates a new claim
// 1.- Creates document
// 2.- Signature proof
// 3.- MerkelTree proof
func (c *claim) Save(ctx context.Context, req *ports.CreateClaimRequest) (*domain.Claim, error) {
 claim, err := c.CreateCredential(ctx, req)
 if err != nil {
  return nil, err
 }
 claim.ID, err = c.icRepo.Save(ctx, c.storage.Pgx, claim)
 if err != nil {
  return nil, err
 }
 if req.SignatureProof {
  err = c.publisher.Publish(ctx, event.CreateCredentialEvent, &event.CreateCredential{CredentialIDs: []string{claim.ID.String()}, IssuerID: req.DID.String()})
  if err != nil {
   log.Error(ctx, "publish CreateCredentialEvent", "err", err.Error(), "credential", claim.ID.String())
  }
 }

 return claim, nil
}
```

其实对应的 RHS 的 path 或者说 key 就是这段代码

<https://github.com/0xPolygonID/issuer-node/blob/84bcb60ace21783051ad8787f35f3585dfbdc70e/internal/core/services/claims.go#L129>

```golang
// CreateCredential - Create a new Credential, but this method doesn't save it in the repository.
func (c *claim) CreateCredential(ctx context.Context, req *ports.CreateClaimRequest) (*domain.Claim, error) {
	if err := c.guardCreateClaimRequest(req); err != nil {
		log.Warn(ctx, "validating create claim request", "req", req)
		return nil, err
	}

	var nonce uint64
	var err error
	if req.RevNonce != nil {
		nonce = *req.RevNonce
	} else {
		nonce, err = rand.Int64()
	}
	if err != nil {
		log.Error(ctx, "create a nonce", "err", err)
		return nil, err
	}

  ...
  ...

}
```

注意其中这段

```go
	if req.RevNonce != nil {
		nonce = *req.RevNonce
	} else {
		nonce, err = rand.Int64()
	}
```

最后通过下面的 Set/Get 成为了 RHS 的 path/key

```go
// SetRevocationNonce sets claim's revocation nonce
func (c *Claim) SetRevocationNonce(nonce uint64) {
	binary.LittleEndian.PutUint64(c.value[0][:8], nonce)
}

// GetRevocationNonce returns revocation nonce
func (c *Claim) GetRevocationNonce() uint64 {
	return binary.LittleEndian.Uint64(c.value[0][:8])
}
```

### 疑问

我的疑问是，如果 RevNonce 重复了， 会出现什么问题?

Update 2024-05-19

官方答复如下: <https://github.com/0xPolygonID/issuer-node/issues/663>
