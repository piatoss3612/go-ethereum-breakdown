# types 패키지

- 이더리움 프로토콜을 구현하는 데 필요한 핵심 타입들이 정의되어 있습니다.

## block.go

- 블록, 블록 헤더, 블록 바디를 정의한 파일입니다.

### Header 

| 필드명 | 설명 | 크기 | json | 특이사항 |
|---|---| ---|---| ---|
| ParentHash | 이전 블록의 해시 | 32 bytes | parentHash |
| UncleHash | 엉클 블록의 해시 | 32 bytes | sha3Uncles |
| Coinbase | 블록 생성자의 주소 | 20 bytes | miner |
| Root | 상태 머클 패트리샤 트리의 머클루트 | 32 bytes | stateRoot |
| TxHash | 트랜잭션의 머클루트 | 32 bytes | transactionsRoot |
| ReceiptHash | 트랜잭션 영수증의 머클루트 | 32 bytes | receiptsRoot |
| Bloom | 블룸필터 | 256 bytes | logsBloom |
| Difficulty | 난이도 | ?? bytes | difficulty |
| Number | 블록 번호 | ?? bytes | number |
| GasLimit | 블록의 gas limit | ?? bytes | gasLimit |
| GasUsed | 블록에서 사용된 gas | ?? bytes | gasUsed |
| Time | 블록 생성 시간 | ?? bytes | timestamp |
| Extra | 블록의 extra data | ?? bytes | extraData |
| MixDigest | 믹스해시 | 32 bytes | mixHash |
| Nonce | 블록의 nonce | 8 bytes | nonce |
| BaseFee | 블록의 base fee | ?? bytes | baseFeePerGas | EIP-1559에서 추가(optional) |
| WithdrawalHash | validator의 인출 요청 시 발생하는 트랜잭션의 머클루트 | 32 bytes | withdrawalHash | EIP-4895에서 추가(optional) |
| BlobGasUsed | 블록에서 사용된 blob gas의 합 | ?? bytes | blobGasUsed | EIP-4844에서 추가(optional) |
| ExcessBlobGas | 블록 당 타깃 사용량을 초과한 blob gas의 양 | ?? bytes | excessBlobGas | EIP-4844에서 추가(optional) |
| ParentBeaconRoot | 이전 비콘 블록의 머클루트 | 32 bytes | parentBeaconRoot | EIP-4788에서 추가(optional) |

- 블록 해시는 RLP 인코딩된 블록 헤더의 keccak256 해시입니다.
- `?? bytes`는 RLP 인코딩에 대해 공부한 후에 다시 살펴보겠습니다.

### Body

| 필드명 | 설명 | 특이사항 |
|---|---| ---|
| Transactions | 트랜잭션 목록 | |
| Uncles | 엉클 블록 목록 | |
| Withdrawals | 출금 작업 목록 | EIP-4895에서 추가(optional) |

### Block

| 필드명 | 설명 | 특이사항 |
|---|---| ---|
| header | 블록 헤더 | |
| uncles | 엉클 블록 목록 | |
| transactions | 트랜잭션 목록 | |
| withdrawals | 출금 작업 목록 | EIP-4895에서 추가(optional) |
| hash | 블록 해시 | 처음 호출 시 계산되고 이후에는 캐싱됩니다. |
| size | 블록의 크기 | 처음 호출 시 계산되고 이후에는 캐싱됩니다. |
| ReceivedAt | 블록이 받아들여진 시간 | 피어 간 블록 릴레이에 사용됩니다. |
| ReceivedFrom | 블록을 보낸 피어의 정보 | 피어 간 블록 릴레이에 사용됩니다. |

#### 블록 불변성 규칙

- 블록 생성 시에 입력되는 모든 값은 복사되어 블록에 저장됩니다. 따라서 입력값이 변경되더라도 블록에 저장된 값은 변경되지 않습니다.
- 블록의 헤더는 블록 해시와 크기에 영향을 미치기 때문에 항상 복사하여 사용합니다.
- 새로운 바디 데이터가 블록에 추가될 때는 블록의 얕은 복사본을 만들어 사용합니다.
- 블록의 바디는 블록 해시와 크기에 영향을 미치지 않고 비용이 많이 들기 때문에 참조하여 사용합니다.

### gen_header_json.go

- 블록 헤더의 JSON 인코딩/디코딩을 위한 코드가 정의되어 있습니다. 

```go
//go:generate go run github.com/fjl/gencodec -type Header -field-override headerMarshaling -out gen_header_json.go
```

- [github.com/fjl/gencodec](https://github.com/fjl/gencodec)

### gen_header_rlp.go

- 블록 헤더의 RLP 인코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run ../../rlp/rlpgen -type Header -out gen_header_rlp.go
```

- [github.com/ethereum/go-ethereum/rlp/rlpgen](https://github.com/ethereum/go-ethereum/tree/master/rlp/rlpgen)

---

## bloom9.go

- 블룸필터를 정의한 파일입니다.
- 블록 헤더의 `Bloom` 필드에 사용되는 로그 블룸필터는 블록에 포함된 트랜잭션의 로그에 대한 빠른 검색을 위해 사용됩니다.
- 비트 코인의 블룸 필터는 관심사를 직접 드러내는 것을 방지하기 위해 프라이버시 보호를 위한 용도로 사용됩니다.

```go
const (
	// BloomByteLength는 블록 헤더의 로그 블룸에 사용되는 바이트 수를 나타냅니다.
	BloomByteLength = 256

	// BloomBitLength는 헤더 로그 블룸에 사용되는 비트 수를 나타냅니다.
	BloomBitLength = 8 * BloomByteLength
)

// Bloom은 2048 비트 블룸 필터를 나타냅니다.
type Bloom [BloomByteLength]byte
```

```go
// bloomValues는 주어진 데이터에 대해 설정할 바이트 (인덱스-값 쌍)를 반환합니다.
// hashbuf는 6바이트 이상의 임시 버퍼여야 합니다.
func bloomValues(data []byte, hashbuf []byte) (uint, byte, uint, byte, uint, byte) {
	sha := hasherPool.Get().(crypto.KeccakState) // keccak256 해시 함수를 풀에서 가져옵니다.
	sha.Reset()                                  // 해시 함수를 초기화합니다.
	sha.Write(data)                              // 데이터를 해싱합니다. (한 번만 해싱합니다.)
	sha.Read(hashbuf)                            // 해시를 읽습니다. (Sum보다 Read가 더 빠릅니다.)
	hasherPool.Put(sha)                          // 해시 함수를 풀에 반환합니다.
	// 필터에 추가되는 비트 자리를 구합니다. (1을 0~7만큼 왼쪽으로 시프트한 값)
	v1 := byte(1 << (hashbuf[1] & 0x7))
	v2 := byte(1 << (hashbuf[3] & 0x7))
	v3 := byte(1 << (hashbuf[5] & 0x7))
	// 데이터를 필터에 추가하기 위해 OR 연산할 바이트의 인덱스
	i1 := BloomByteLength - uint((binary.BigEndian.Uint16(hashbuf)&0x7ff)>>3) - 1 // v1의 바이트 인덱스는 hashbuf를 uint16 빅 엔디언으로 읽은 값의 하위 11비트를 3만큼 오른쪽으로 시프트한 값을 uint로 변환한 값에서 1을 뺀 값입니다.
	i2 := BloomByteLength - uint((binary.BigEndian.Uint16(hashbuf[2:])&0x7ff)>>3) - 1 
	i3 := BloomByteLength - uint((binary.BigEndian.Uint16(hashbuf[4:])&0x7ff)>>3) - 1

	return i1, v1, i2, v2, i3, v3
}
```

- 정확히 어떤 알고리즘을 사용하는지는 모르겠습니다. (아시는 분은 알려주세요.)

---

## hashes.go

- 비어있는 트리의 루트를 상황에 맞게 사용할 수 있게 변수로 정의해둔 파일입니다.

---

## hashing.go

- keccak256 해시 함수의 인스턴스를 재사용하기 위한 풀과 머클루트를 계산하기 위해 구현해야 하는 인터페이스를 정의한 파일입니다.
- 해시 함수 및 임시 버퍼 풀은 `sync.Pool`을 사용하여 구현되어 있습니다.
- 풀에서 가져온 객체는 사용 후에는 반드시 풀에 반환해야 합니다.
- `TrieHasher`, `DerivableList`는 머클루트를 계산하기 위해 구현해야 하는 인터페이스입니다.

- `StackTrie`가 무엇인지 의문. 이해하기 어려운 주석이 달려있습니다.

---

## log.go

- 이더리움의 이벤트 로그 타입을 정의한 파일입니다.

### 컨센서스 필드: 이더리움 황서에 정의된 필드

| 필드명 | 설명 | json | 특이사항 |
|---|---| ---| ---|
| Address | 이벤트 로그를 생성한 컨트랙트의 주소 | address | |
| Topics | 이벤트 로그의 토픽 | topics | |
| Data | 이벤트 로그의 데이터 | data | abi 인코딩된 데이터 |

### 파생 필드: 노드 구현체에서 추가한 필드

| 필드명 | 설명 | json | 특이사항 |
|---|---| ---| ---|
| BlockNumber | 이벤트 로그가 포함된 블록의 번호 | blockNumber | |
| TxHash | 이벤트 로그가 포함된 트랜잭션의 해시 | transactionHash | |
| TxIndex | 이벤트 로그가 포함된 트랜잭션의 인덱스 | transactionIndex | |
| BlockHash | 이벤트 로그가 포함된 블록의 해시 | blockHash | |
| Index | 이벤트 로그의 인덱스 | logIndex | |
| Removed | 이벤트 로그가 제거되었는지 여부 | removed | 체인 재구성으로 인해 이벤트 로그가 제거되었을 때 true로 설정됩니다. |

### gen_log_json.go

- 로그의 JSON 인코딩/디코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run github.com/fjl/gencodec -type Log -field-override logMarshaling -out gen_log_json.go
```

- [github.com/fjl/gencodec](https://github.com/fjl/gencodec)

### gen_log_rlp.go

- 로그의 RLP 인코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run ../../rlp/rlpgen -type Log -out gen_log_rlp.go
```

- [github.com/ethereum/go-ethereum/rlp/rlpgen](https://github.com/ethereum/go-ethereum/tree/master/rlp/rlpgen)

---

## receipt.go

- 트랜잭션 결과에 대한 영수증을 정의한 파일입니다.
- 컨센서스 필드만으로 RLP 인코딩/디코딩을 하는 컨센서스 방식
- 컨센서스 필드에서 블룸 필터를 제외한 필드만으로 RLP 인코딩하고 디코딩할 때 재계산하는 스토리지 방식
- 왜 두 가지 방식으로 나누어져 있는지는 모르겠습니다.

### 컨센서스 필드: 이더리움 황서에 정의된 필드

| 필드명 | 설명 | json | 특이사항 |
|---|---| ---| ---|
| Type | 영수증의 타입 | type | |
| PostState | 트랜잭션 실행 후의 상태 | postState | |
| Status | 트랜잭션의 실행 결과 | status | 0: 실패, 1: 성공 |
| CumulativeGasUsed | 블록에서 사용된 gas의 누적량 | cumulativeGasUsed | |
| Bloom | 로그 블룸필터 | logsBloom | |
| Logs | 이벤트 로그 목록 | logs | |

### 파생 필드: 노드 구현체에서 추가한 필드

| 필드명 | 설명 | json | 특이사항 |
|---|---| ---| ---|
| TxHash | 트랜잭션의 해시 | transactionHash | |
| ContractAddress | 컨트랙트 생성 트랜잭션의 경우 생성된 컨트랙트의 주소 | contractAddress | |
| GasUsed | 트랜잭션 실행 시 사용된 gas의 양 | gasUsed | |
| EffectiveGasPrice | 트랜잭션의 effective gas price | effectiveGasPrice | |
| BlobGasUsed | 트랜잭션 실행 시 사용된 blob gas의 양 | blobGasUsed | |
| BlobGasPrice | 트랜잭션의 blob gas price | blobGasPrice | |
| BlockHash | 트랜잭션이 포함된 블록의 해시 | blockHash | |
| BlockNumber | 트랜잭션이 포함된 블록의 번호 | blockNumber | |
| TransactionIndex | 트랜잭션의 인덱스 | transactionIndex | |

### gen_receipt_json.go

- 영수증의 JSON 인코딩/디코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run github.com/fjl/gencodec -type Receipt -field-override receiptMarshaling -out gen_receipt_json.go
```

- [github.com/fjl/gencodec](https://github.com/fjl/gencodec)

---

## state_account.go

- 이더리움 계정을 정의한 파일입니다.
- 계정에는 slim 형식과 full-consensus 형식이 있습니다.

```go
// StateAccount는 이더리움 컨센서스 계정입니다.
// 이 객체들은 메인 계정 트라이에 저장됩니다.
type StateAccount struct {
	Nonce    uint64      // 계정의 nonce
	Balance  *big.Int    // 계정의 잔액
	Root     common.Hash // 스토리지 트라이의 머클루트
	CodeHash []byte      // EVM 코드 해시 (Externally Owned Account는 nil)
}
```

### gen_account_rlp.go

- 계정의 RLP 인코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run ../../rlp/rlpgen -type StateAccount -out gen_account_rlp.go
```

- [github.com/ethereum/go-ethereum/rlp/rlpgen](https://github.com/ethereum/go-ethereum/tree/master/rlp/rlpgen)

---

## transaction.go

- 트랜잭션을 정의한 파일입니다.

### 트랜잭션 타입

- 트랜잭션 타입은 4가지가 있습니다. (Legacy, AccessList, DynamicFee, Blob)

```go
const (
	LegacyTxType     = 0x00 // Legacy
	AccessListTxType = 0x01 // EIP-2930
	DynamicFeeTxType = 0x02 // EIP-1559
	BlobTxType       = 0x03 // EIP-4844
)
```

### TxData 인터페이스

```go
// TxData는 트랜잭션의 기본 데이터를 나타내기 위한 인터페이스입니다.
//
// 이 인터페이스는 DynamicFeeTx, LegacyTx, AccessListTx에 의해 구현됩니다.
type TxData interface {
	txType() byte // 트랜잭션 타입 ID를 반환합니다.
	copy() TxData // 깊은 복사본을 만들어 새로운 TxData를 반환합니다.

	chainID() *big.Int
	accessList() AccessList
	data() []byte
	gas() uint64
	gasPrice() *big.Int
	gasTipCap() *big.Int
	gasFeeCap() *big.Int
	value() *big.Int
	nonce() uint64
	to() *common.Address

	rawSignatureValues() (v, r, s *big.Int)
	setSignatureValues(chainID, v, r, s *big.Int)

	// effectiveGasPrice는 트랜잭션이 지불하는 가스 가격을 계산합니다. 트랜잭션이 포함된 블록의 baseFee가 주어집니다.
	//
	// 다른 TxData 메서드와 달리, 반환된 *big.Int는 계산된 값의 독립적인 복사본이어야 합니다.
	// 즉, 호출자는 결과를 변경할 수 있습니다. 메서드 구현은 'dst'를 사용하여 결과를 저장할 수도 있습니다.
	effectiveGasPrice(dst *big.Int, baseFee *big.Int) *big.Int

	encode(*bytes.Buffer) error
	decode([]byte) error
}
```

### transaction_marshaling.go

- 트랜잭션의 JSON 인코딩/디코딩을 위한 코드가 정의되어 있습니다.

---

## tx_legacy.go

- Legacy 트랜잭션을 정의한 파일입니다.
- `TxData` 인터페이스를 구현합니다.

```go
// LegacyTx는 근본 이더리움 트랜잭션 데이터입니다.
type LegacyTx struct {
	Nonce    uint64          // 발신자 계정의 nonce
	GasPrice *big.Int        // 가스당 wei
	Gas      uint64          // 가스 한도
	To       *common.Address `rlp:"nil"` // 수신자의 주소. nil이면 컨트랙트 생성 트랜잭션
	Value    *big.Int        // wei 단위의 이더량
	Data     []byte          // 컨트랙트 생성 트랜잭션의 경우 생성자의 바이트코드. 그 외의 경우 호출 데이터
	V, R, S  *big.Int        // 서명 값
}
```

---

## tx_access_list.go

- EIP-2930에서 사용되는 접근 목록과 접근 목록 트랜잭션을 정의한 파일입니다.

```go
// AccessList은 EIP-2930에 정의된 접근 목록입니다.
type AccessList []AccessTuple

// AccessTuple은 접근 목록의 요소입니다.
type AccessTuple struct {
	Address     common.Address `json:"address"     gencodec:"required"`
	StorageKeys []common.Hash  `json:"storageKeys" gencodec:"required"`
}
```

```go
// AccessListTx는 EIP-2930 접근 목록 트랜잭션의 데이터입니다.
type AccessListTx struct {
	ChainID    *big.Int        // 대상 체인 ID
	Nonce      uint64          // 발신자 계정의 nonce
	GasPrice   *big.Int        // 가스당 wei
	Gas        uint64          // 가스 한도
	To         *common.Address `rlp:"nil"` // 수신자의 주소. nil이면 컨트랙트 생성 트랜잭션
	Value      *big.Int        // wei 단위의 이더량
	Data       []byte          // 컨트랙트 생성 트랜잭션의 경우 생성자의 바이트코드. 그 외의 경우 호출 데이터
	AccessList AccessList      // 접근 목록
	V, R, S    *big.Int        // 서명 값
}
```

### gen_access_tuple.go

- 접근 목록의 요소를 JSON 인코딩/디코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run github.com/fjl/gencodec -type AccessTuple -out gen_access_tuple.go
```

- [github.com/fjl/gencodec](https://github.com/fjl/gencodec)

---

## tx_dynamic_fee.go

- EIP-1559에서 사용되는 gas fee cap과 gas tip cap을 필드로 추가된 트랜잭션을 정의한 파일입니다.

```go
// DynamicFeeTx는 EIP-1559 트랜잭션을 나타냅니다.
type DynamicFeeTx struct {
	ChainID    *big.Int
	Nonce      uint64
	GasTipCap  *big.Int // a.k.a. maxPriorityFeePerGas
	GasFeeCap  *big.Int // a.k.a. maxFeePerGas
	Gas        uint64
	To         *common.Address `rlp:"nil"` // nil이면 컨트랙트 생성 트랜잭션
	Value      *big.Int
	Data       []byte
	AccessList AccessList

	// 서명 값
	V *big.Int `json:"v" gencodec:"required"`
	R *big.Int `json:"r" gencodec:"required"`
	S *big.Int `json:"s" gencodec:"required"`
}
```

---

## tx_blob.go

- EIP-4844에서 사용되는 blob gas와 sidecar가 필드로 추가된 트랜잭션을 정의한 파일입니다.


```go
// BlobTx는 EIP-4844 트랜잭션을 나타냅니다.
type BlobTx struct {
	ChainID    *uint256.Int
	Nonce      uint64
	GasTipCap  *uint256.Int // a.k.a. maxPriorityFeePerGas
	GasFeeCap  *uint256.Int // a.k.a. maxFeePerGas
	Gas        uint64
	To         common.Address
	Value      *uint256.Int
	Data       []byte
	AccessList AccessList
	BlobFeeCap *uint256.Int // a.k.a. maxFeePerBlobGas
	BlobHashes []common.Hash

	// blob 트랜잭션은 선택적으로 blob을 포함할 수 있습니다. BlobTx가 서명을 위해 트랜잭션을 생성하는 데 사용될 때 이 필드를 설정해야만 합니다.
	Sidecar *BlobTxSidecar `rlp:"-"`

	// 서명 값
	V *uint256.Int `json:"v" gencodec:"required"`
	R *uint256.Int `json:"r" gencodec:"required"`
	S *uint256.Int `json:"s" gencodec:"required"`
}

// BlobTxSidecar는 blob 트랜잭션의 blob을 포함합니다.
type BlobTxSidecar struct {
	Blobs       []kzg4844.Blob       // blob 풀이 필요한 blob
	Commitments []kzg4844.Commitment // blob 풀이 필요한 Commitments
	Proofs      []kzg4844.Proof      // blob 풀이 필요한 Proofs
}
```

---

## transaction_signing.go

- 트랜잭션에 서명하거나 서명자를 확인하는 코드가 정의되어 있습니다.

### 서명 함수

```go
// SignTx는 주어진 서명자와 개인 키를 사용하여 트랜잭션에 서명합니다.
func SignTx(tx *Transaction, s Signer, prv *ecdsa.PrivateKey) (*Transaction, error) {
	h := s.Hash(tx)                    // 서명 해시 생성 (Signer에 따라 다르게 생성됨)
	sig, err := crypto.Sign(h[:], prv) // 개인 키로 서명 (직렬화된 서명 데이터 반환)
	if err != nil {
		return nil, err
	}
	return tx.WithSignature(s, sig) // 트랜잭션에 서명 데이터 추가 (V, R, S 값 설정 + 서명자의 체인 ID 설정)
}
```

### Signer 인터페이스

```go
// Signer는 트랜잭션 서명 처리 기능을 캡슐화합니다. 이 타입의 이름은 약간 오해의 소지가 있습니다.
// 왜냐하면 Signer는 실제로 서명하지 않고 서명을 검증하고 처리하기 위한 것이기 때문입니다.
//
// 참고로 이 인터페이스는 안정적인 API가 아니며 새로운 프로토콜 규칙을 수용하기 위해 언제든지 변경될 수 있습니다.
type Signer interface {
	// Sender는 트랜잭션의 발신자 주소를 반환합니다.
	Sender(tx *Transaction) (common.Address, error)

	// SignatureValues는 주어진 서명에 해당하는 원시 R, S, V 값을 반환합니다.
	SignatureValues(tx *Transaction, sig []byte) (r, s, v *big.Int, err error)
	ChainID() *big.Int

	// Hash는 '서명 해시'를 반환합니다. 즉, 개인 키를 사용하여 서명되기 전의 트랜잭션 해시입니다.
	// 이 해시는 트랜잭션을 고유하게 식별하지는 않습니다.
	Hash(tx *Transaction) common.Hash

	// Equal은 주어진 서명자가 수신자와 동일한지 여부를 반환합니다.
	Equal(Signer) bool
}
```

### Signer 인터페이스를 구현하는 타입

- 각 구현체는 하위 버전의 구현체를 임베딩하여 상위 버전의 구현체를 만듭니다.

```go
type cancunSigner struct{ londonSigner }

func NewCancunSigner(chainId *big.Int) Signer {
	return cancunSigner{londonSigner{eip2930Signer{NewEIP155Signer(chainId)}}}
}

type londonSigner struct{ eip2930Signer }

func NewLondonSigner(chainId *big.Int) Signer {
	return londonSigner{eip2930Signer{NewEIP155Signer(chainId)}}
}

type eip2930Signer struct{ EIP155Signer }

func NewEIP2930Signer(chainId *big.Int) Signer {
	return eip2930Signer{NewEIP155Signer(chainId)}
}

type EIP155Signer struct {
	chainId, chainIdMul *big.Int
}

func NewEIP155Signer(chainId *big.Int) EIP155Signer {
	if chainId == nil {
		chainId = new(big.Int)
	}
	return EIP155Signer{
		chainId:    chainId,
		chainIdMul: new(big.Int).Mul(chainId, big.NewInt(2)),
	}
}
```

- 단, `HomesteadSigner`와 `FrontierSigner`는 별도로 구현되어 있습니다.

```go
type HomesteadSigner struct{ FrontierSigner }

type FrontierSigner struct{}
```

### 서명 해시 생성 방식의 차이

<img src="./sighash.png" >

---

## withdrawal.go

- EIP-4895에서 사용되는 출금 작업을 정의한 파일입니다.

```go
// Withdrawal은 합의 레이어로부터 검증자의 출금 작업을 나타냅니다.
type Withdrawal struct {
	Index     uint64         `json:"index"`          // 합의 레이어에 의해 발행된 단조 증가식 식별자
	Validator uint64         `json:"validatorIndex"` // 출금과 관련된 검증자의 인덱스
	Address   common.Address `json:"address"`        // 출금된 이더가 전송되는 주소
	Amount    uint64         `json:"amount"`         // 출금액 (Gwei 단위)
}
```

### gen_withdrawal_json.go

- 출금 작업의 JSON 인코딩/디코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run github.com/fjl/gencodec -type Withdrawal -field-override withdrawalMarshaling -out gen_withdrawal_json.go
```

- [github.com/fjl/gencodec](https://github.com/fjl/gencodec)

### gen_withdrawal_rlp.go

- 출금 작업의 RLP 인코딩을 위한 코드가 정의되어 있습니다.

```go
//go:generate go run ../../rlp/rlpgen -type Withdrawal -out gen_withdrawal_rlp.go
```

- [github.com/ethereum/go-ethereum/rlp/rlpgen](https://github.com/ethereum/go-ethereum/tree/master/rlp/rlpgen)

---

## 타입들 간의 관계 정리

![types](./relations.png)

---

## 읽어보기

- [EIP-155](https://eips.ethereum.org/EIPS/eip-155)
- [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559)
- [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718)
- [EIP-2930](https://eips.ethereum.org/EIPS/eip-2930)
- [EIP-4895](https://eips.ethereum.org/EIPS/eip-4895)
- [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844)
- [EIP-4788](https://eips.ethereum.org/EIPS/eip-4788)
