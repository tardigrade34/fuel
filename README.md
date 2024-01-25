# Fuel İstemcisi

## Derleme

### Sistem Gereksinimleri

Clang dahil olmak üzere birkaç sistem gereksinimi bulunmaktadır.

#### MacOS

```bash
brew update
brew install cmake
brew install protobuf
```

#### Debian

```bash
apt update
apt install -y cmake pkg-config build-essential git clang libclang-dev protobuf-compiler
```

#### Arch

```bash
pacman -Syu --needed --noconfirm cmake gcc pkgconf git clang protobuf-compiler
```

### Derleme

`xtask` kullanmanızı öneririz:

```sh
cargo xtask build
```

Bu, `cargo build`'ı çalıştıracak ve istemci için GraphQL şemasını yeniden oluşturmak gibi diğer özel derleme işlemlerimizi de içerecektir.

### Test Etme

[ci_checks.sh](ci_checks.sh) betik dosyası, tüm CI kontrollerini, testleri çalıştırmayı da içeren şekilde çalıştırmak için kullanılabilir.

```shell
source ci_checks.sh
```

Betik, önceden yüklenmiş araçları gerektirir. Daha fazla bilgi için şunu çalıştırın:

```shell
cat ci_checks.sh
```

## Çalıştırma

Hizmeti `fuel-core run` komutuyla başlatabilirsiniz. Çalıştırmak için kullanılabilen seçenekler `help` seçeneği aracılığıyla erişilebilir:

```console
$ ./target/debug/fuel-core run --help

KULLANIM:
    fuel-core run [SEÇENEKLER]

SEÇENEKLER:
        --chain <CHAIN_CONFIG>
            Dahili bir yapılandırmaya bir takma ad veya bir JSON dosyası yolunu belirtin [varsayılan:
            local_testnet]
        ...
```

Birçok geliştirme amaçlı olarak, kalıcı olmayan bir duruma sahip olmak faydalıdır ve `db-type` seçeneği aşağıdaki örnekte olduğu gibi `in-memory` olarak ayarlanabilir.

### Örnek

```console
$ ./target/debug/fuel-core run --db-type in-memory
2023-06-13T12:45:22.860536Z  INFO fuel_core::cli::run: 230: Blok üretim modu: Anında
2023-06-13T12:38:47.059783Z  INFO fuel_core::cli::run: 310: Fuel Core sürümü v0.18.1
2023-06-13T12:38:47.078969Z  INFO new{name=fuel-core}:_commit_result{block_id=b1807ca9f2eec7e459b866ecf69b68679fc6b205a9a85c16bd4943d1bfc6fb2a height=0 tx_status=[]}: fuel_core_importer::importer: 231: Blok onaylandı
2023-06-13T12:38:47.097777Z  INFO new{name=fuel-core}: fuel_core::graphql_api::service: 208: GraphQL sağlayıcısını 127.0.0.1:4000'e bağlama
```

Yerel düğümünüzde blok üretimini devre dışı bırakmak için `--poa-instant=false` seçeneğini ayarlayın.

### Örnek

```console
$ ./target/debug/fuel-core run --poa-instant=false
2023-06-13T12:44:12.857763Z  INFO fuel_core::cli::run: 232: Blok üretimi devre dışı bırakıldı
```

### Sorun Giderme

#### Yayınlama

Tüm kafesleri otomatik olarak yayınlamak için [`publish-crates`](https://github.com/katyo/publish-crates) eylemini kullanıyoruz.

Yayınlama ile ilgili sorunlar yaşıyorsanız, bunu [`act`](https://github.com/nektos/act) ile yerel olarak sorun gidermeyi düşünebilirsiniz.

```shell
act release -s GITHUB_TOKEN=<YOUR_GITHUB_TOKEN> -j publish-crates-check --container-architecture linux/amd64 --reuse
```

Bu, GitHub'a istek göndermek için GitHubToken'a ihtiyaç duyar. Bunu [bu](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) yönergeyi kullanarak oluşturabilirsiniz.

#### Güncellenmemiş veritabanı

Eğer şu gibi bir hata ile karşılaşırsanız:

```console
thread 'main' panicked at 'unable to open database: DatabaseError(Error { message: "Invalid argument: Column families not opened: column-11, column-10, column-9, column-8, column-7, column-6, column-5, column-4, column-3, column-2, column-1, column-0" })', fuel-core/src/main.rs:23:66
```

Yerel veritabanınızı temizlemek için şu komutu kullanın: `rm -rf ~/.fuel/db`

#### Dosya açıkl

ığı sınırları

Bazı MacOS sürümlerinde varsayılan dosya açıklığı sınırı oldukça düşük olabilir, bu da `Too many open files` gibi IO hatalarına veya hatta RocksDB bu sorunlarla karşılaştığında `fatal runtime error: Rust cannot catch foreign exceptions` gibi hatalara neden olabilir. Açık dosya sınırını artırmak için aşağıdaki komutu kullanın. Bu yalnızca mevcut kabuk oturumu üzerinde etki eder, bu nedenle bunu `~/.zshrc`'ye eklemeyi düşünün.

```bash
ulimit -n 10240
```

#### Günlük seviyesi

Servis, `RUST_LOG` ortam değişkenine dayanır. Daha fazla bilgi için [EnvFilter examples](https://docs.rs/tracing-subscriber/latest/tracing_subscriber/filter/struct.EnvFilter.html#examples) paketini kontrol edin.

İnsan günlüğü tutma, `HUMAN_LOGGING=false` ortam değişkeni ile devre dışı bırakılabilir.

## Hata Ayıklama

Yerel bir düğümün hata ayıklanması hakkında genel bir bilgi için [hata ayıklama kılavuzunu](docs/developers/debugging.md) inceleyin.

## Docker & Kubernetes

```sh
# Docker Görüntüsü Oluştur
docker build -t fuel-core . -f deployment/Dockerfile

# Docker Görüntüsünü Sil
docker image rm fuel-core

# Kubernetes Hacmi, Dağıtımı ve Servisi Oluştur
kubectl create -f deployment/fuel-core.yml

# Kubernetes Hacmi, Dağıtımı ve Servisi Sil
kubectl delete -f deployment/fuel-core.yml
```

## GraphQL Servisi

İstemcinin işlevselliği, GraphQL sorgularını bekleyen bir hizmet noktası aracılığıyla kullanılabilir.

### İşlem Yürütücüsü

İşlem yürütücü şu anda anında blok üretimi gerçekleştirir. Değişiklikler varsayılan olarak RocksDB'ye kalıcı olarak yazılır.

- Hizmet noktası: `/graphql`
- Şema (derlendikten sonra kullanılabilir): `crates/client/assets/schema.sdl`

Servis, hex kodlu ikili formatında bir [Transaction](https://github.com/FuelLabs/fuel-vm/tree/master/fuel-tx) nesnesini alan `submit` adlı bir mutasyon bekler, [burada belirtildiği gibi](https://github.com/FuelLabs/fuel-specs/blob/master/src/tx-format/transaction.md).

### cURL Örneği

Bu örnek, aşağıdaki [ASM](https://github.com/FuelLabs/fuel-vm/tree/master/fuel-asm) dizisini temsil eden bir betiği yürütecek bir komut dosyasını çalıştırır:

```rs
ADDI(0x10, RegId::ZERO, 0xca),
ADDI(0x11, RegId::ZERO, 0xba),
LOG(0x10, 0x11, RegId::ZERO, RegId::ZERO),
RET(RegId::ONE),
```

```console
$ cargo run --bin fuel-core-client -- transaction submit \
"{\"Script\":{\"script_gas_limit\":1000000,\"policies\":{\"bits\":\"GasPrice\",\"values\":[0,0,0,0]},\"maturity\":0,\"script\":[80,64,0,202,80,68,0,186,51,65,16,0,36,4,0,0],\"script_data\":[],\"inputs\":[
{
  \"CoinSigned\": {
    \"utxo_id\": {
      \"tx_id\": \"c49d65de61cf04588a764b557d25cc6c6b4bc0d7429227e2a21e61c213b3a3e2\",
      \"output_index\": 0
    },
    \"owner\": \"f1e92c42b90934aa6372e30bc568a326f6e66a1a0288595e6e3fbd392a4f3e6e\",
    \"amount\": 10599410012256088338,
    \"asset_id\": \"2cafad611543e0265d89f1c2b60d9ebf5d56ad7e23d9827d6b522fd4d6e44bc3\",
    \"tx_pointer\": {
      \"block_height\": 0,
      \"tx_index\": 0
    },
    \"witness_index\": 0,
    \"maturity\": 0,
    \"predicate_gas_used\": null,
    \"predicate\": null,
    \"predicate_data\": null
  }
}],\"outputs\":[],\"witnesses\":[{
  \"data\": [
    150,31,98,53,6,239,255,243,45,35,182,26,129,152,46,95,45,211,114,58,51,40,129,194,97,14,181,70,190,37,106,223,170,174,221,230,87,239,67,224,100,137,25,249,193,14,184,195,15,85,156,82,91,78,91,80,126,168,215,170,139,48,19,5
  ]
}],\"receipts_root\":\"0x6114142d12e0f58cfb8c72c270cd0535944fb1ba763dce83c17e882c482224a2\"}}"
```
