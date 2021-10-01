# Lux AI Season 1 Kits

このフォルダには、Lux AIチームがLux AI Challenge Season 1のために提供したすべての公式キットが含まれています。

各スターターキットのフォルダの中には、READMEと、競技に必要なすべてのツールを提供する`simple`というフォルダが入っています。READMEの文書をよく読むようにしてください。デバッグの際には、`console.error`や`print("hello", file=sys.stderr)`などの標準エラーにログを取ることができます。これは各エージェントの試合のerrorlogsフォルダに記録・保存され、競技サーバーにも記録されます。また、[debug annotations](#Annotate)コマンドを使って、リプレイに絵を描くこともできますのでお試しください。

## API

このセクションでは、各キットが準拠している一般的なAPIについて詳しく説明します。ここではPythonの規約に従っていますが、他のキットも全く同じで、構文や命名規則が異なるだけです。個々の違いについては、各キットのコメントに記載されています。

いくつかのメソッドは**アクション**を生成するものです。例えば、[Unit](#Unit)の移動、リソースの移動、ユニットの構築などです。これらのアクションメソッドは常に文字列を返すので、`actions`配列に追加する必要があり、スターターキットにはその方法の例があります。

### game_state

`game_state` オブジェクトは、現在のターン `game_state.turn` におけるゲームの現在の状態についての完全な情報を含んでいます。ゲーム内の各エージェントやプレイヤーにはidがあり、自分のボットのidは `game_state.id` となり、他のチームのidは `(game_state.id + 1) % 2` となります。

また、ゲームステートの中には、[GameMap](#GameMap)タイプの`map`と、プレイヤーのチームIDでインデックスされた2つの[Player](#Player)オブジェクトのリストである`players`というネストされたオブジェクトがあります。キットでは、これらのオブジェクトを取得する方法を説明します。このセクションの残りの部分では、キットで使用される各タイプのオブジェクトのプロパティとメソッドの詳細を説明します。

### <u>GameMap</u>

地図は，左上が `(0, 0)`，右下が `(幅 - 1, 高さ - 1)` となるように構成されている。地図は常に正方形である。

Properties:

- `height: int` - マップの高さ（Y軸方向）です。
- `width: int` - マップの幅（x方向）です。
- `map: List[List[Cell]]` - [Cell](#Cell)オブジェクトの2次元配列で、マップの現在の状態を定義します。map[y][x]`は座標(x, y)のセルを表し、`map[0][0]`は左上のセルを表します。

Methods:

- `get_cell_by_pos(pos: Position) -> Cell` - 与えられた位置の[Cell](#Cell)を返します。
- `get_cell(x: int, y: int) -> Cell` - 指定された x, y 座標で [Cell](#Cell) を返します。

### <u>Position</u>

Properties:

- `x: int` - [Position](#Position)の x 座標
- `y: int` - [Position](#Position)の y 座標

Methods:

- `is_adjacent(pos: Position) -> bool` - この [Position](#Position) が `pos` に隣接している場合は true を返します。それ以外は False

- `equals(pos: Position) -> bool` - x, y 座標を確認して、この [Position](#Position) が他の `pos` オブジェクトと等しい場合は true を返します。それ以外の場合は False を返します。

- `translate(direction: DIRECTIONS, units: int) -> Position` - この [Position](#Position) から `direction` の `units` 倍の方向に進んだときの [Position](#Position) を返します。

- `distance_to(pos: Position) -> float` - この [Position](#Position) から `pos` までの [Manhattan (rectilinear) distance](https://en.wikipedia.org/wiki/Taxicab_geometry) を返します。

- direction_to(target_pos: Position) -> DIRECTIONS` - この[Position](#Position)から1歩進んだ場合に、`target_pos`に最も近づくことができる方向を返します。特に、この[Position](#Position)が `target_pos` と等しい場合には、`DIRECTIONS.CENTER` を返します。これは他のユニットとの衝突の可能性をチェックするものではなく、基本的な経路探索方法として機能することに注意してください。

### <u>Cell</u>

Properties:

- `pos: Position
- `resource: Resource` - この[Cell](#Cell)にあるResourceの詳細を含みます。これは他の言語では `None` または `null` に相当するものと同じかもしれません。この[Cell](#Cell)にリソースがあるかどうかを確認するには、常に関数 `has_resource` を使用する必要があります。
- road: float` - [ユニット](#Unit)がこのタイル上でアクションを行う際に、クールダウンから差し引かれる量です。道路がある場合、その道路が発達しているほど、このクールダウン率の値は高くなります。なお、[ユニット](#Unit)は何かアクションを行うたびに必ず基本のクールダウン量を得る。
- citytile: CityTile` - この[Cell](#Cell)の上にある都市タイルです。他の言語では、[CityTile](#CityTile)がない場合、`none`または`null`に相当します。

Methods:

- `has_resource() -> bool` - この[Cell](#Cell)が枯渇していないResourceを持っている場合はtrueを、そうでない場合はfalseを返します。

### <u>City</u>

Properties:

- cityid: str` - この[City](#City)のIDです。ゲーム内の各[City](#City)のIDはユニークで、新しい都市で再利用されることはありません。
- team: int` - この[City](#City)が所属するチームのIDです。
- fuel: float` - この[City](#City)に蓄えられている燃料です。この燃料はこの[City](#City)にあるすべてのCityTilesが夜のターンに消費します。
- citytiles: list[CityTile]` - 1つの[City](#City)を構成する[CityTile](#CityTile)オブジェクトのリストです。City](#City)とは、隣接するCityTileを介して接続されているすべてのCityTileを指します。

Methods:

- `get_light_upkeep() -> float` - は、[City](#City)のターンごとに光のアップキープを返します。都市](#都市)内の燃料は、夜のターンごとにライトアップキープ分だけ減算される。

### <u>CityTile</u>

Properties:

- `cityid: str` - は、この[CityTile](#CityTile)が属する[City](#City)のIDです。ゲーム内の各[City](#City)のidはユニークであり、新しい都市で再利用されることはありません。
- `team: int` - この[CityTile](#CityTile)が所属するチームのIDです。
- `pos: Position` - この[City]（#City）の地図上の[位置]（#Position）を表示します。
- `cooldown: float` - この[City](#City)の現在のCooldownです。

Methods:

- `can_act() -> bool` - whether this [City](#City) can perform an action this turn, which is when the Cooldown is less than 1

- `research() -> str` - returns the research action

- `build_worker() -> str` - returns the build worker action. When applied and requirements are met, a worker will be built at the [City](#City).

- `build_cart() -> str` - returns the build cart action. When applied and requirements are met, a cart will be built at the [City](#City).

### <u>Unit</u>

Properties:

- `pos: Position` - the [Position](#Position) of this [Unit](#Unit) on the map
- `team: int` - the id of the team this [Unit](#Unit) belongs to.
- `id: str` - the id of this [Unit](#Unit). This is unique and cannot be repeated by any other [Unit](#Unit) or [City](#City)
- `cooldown: float` - the current Cooldown of this [Unit](#Unit). Note that when this is less than 1, the [Unit](#Unit) can perform an action
- `cargo.wood: int` - the amount of wood held by this [Unit](#Unit)
- `cargo.coal: int` - the amount of coal held by this [Unit](#Unit)
- `cargo.uranium: int` - the amount of uranium held by this [Unit](#Unit)

Methods:

- `get_cargo_space_left(): int` - この[Unit](#ユニット)のカーゴに残っているスペースの量を返します。例えば、70個の木材は70個のウランと同じだけのスペースを取りますが、70個のウランは[City](#City)に置かれた場合、木材よりもはるかに多くの燃料を生産します。
- can_build(game_map: GameMap): bool` - [Unit](#Unit)が今いるタイルに[City](#City)を建設できる場合はtrueを、そうでない場合はfalseを返します。それ以外の場合は偽を返します。そのタイルにリソースが存在せず、[Unit](#Unit)のクールダウンが1以下であることを確認する。
- can_act(): bool` - [Unit](#Unit)がアクションを実行できる場合はtrueを返します。それ以外の場合は False を返します。基本的には、[Unit](#Unit)の Cooldown が 1 より小さいかどうかを調べます。
- `move(dir): str` - 移動アクションを返します。適用されると、[Unit](#Unit)は指定された方向に[Unit](#Unit)1つ分移動しますが、他のユニットが邪魔していたり、反対側の都市が存在しないことが条件です。([ユニット](#ユニット)は味方の[都市](#都市)の上では重ねることができる)
- `transfer(dest_id, resourceType, amount): str` - 転送アクションを返します。ターン開始時に両ユニットが隣接している場合、この[ユニット](#ユニット)から選択されたリソースタイプを希望の量だけ、idが`dest_id`の[ユニット](#ユニット)に転送します。(これは、目的地の[ユニット](#ユニット)が他の[ユニット](#ユニット)から資源の譲渡を受けても、その[ユニット](#ユニット)から離れることができることを意味します)
- `build_city(): str` - [City](#City)を構築するアクションを返します。適用されると、[Unit](#Unit)は[City](#City)も資源もない空のタイルで、ワーカーが100ユニットの資源を持っている場合、自分の真下に[City](#City)を建設しようとする。都市の建設に成功した場合、すべての資源は消費される。
- pillage(): str` - ピレッジアクションを返します。適用されると、[Unit](#Unit)は現在上に乗っているタイルを略奪し、道路レベルの0.5を取り除きます。

### <u>Player</u>

特定のチームの特定のプレーヤーに関する情報を含みます。

Properties:

- `team: int` - このプレイヤーのチームIDです。

- `research_points: int` - このプレイヤーのチームが持っているリサーチポイントの現在の合計数です。
- `units: list[Unit]` - このプレイヤーのチームが所有するすべての[Unit](#Unit)のリストです。
- `cities: Dict[str, City]` - [City](#City)のIDと、このプレイヤーのチームが所有する各[City](#City)をマッピングした辞書/マップです。個々のCityTilesを取得するには、[City](#City)の`citytiles`プロパティにアクセスする必要があります。

Methods:

- `researched_coal() - bool` - whether or not this player's team has researched coal and can mine coal.
- `researched_uranium() - bool` - whether or not this player's team has researched uranium and can mine uranium.

### <u>Annotate</u>

アノテーションオブジェクトでは、デバッグモードがオンになっているときにビジュアライザーに表示されるアノテーションコマンドを作成することができます。なお、これらのコマンドは大会サーバーでは削除されますが、ローカルで試合を行う際には見ることができます。

Methods

- `circle(x: int, y: int) -> str` - 円を描くアノテーションアクションを返します。与えられたx, y座標の[Cell](#Cell)を中心に、現在のターンのビジュアライザーに単位サイズの円を描きます。

- `x(x: int, y: int) -> str` - draw Xアノテーションアクションを返します。与えられたx, y座標の[Cell](#Cell)を中心に、現在のターンのビジュアライザーに単位サイズのXを描画します。

- `line(x1: int, y1: int, x2: int, y2: int) -> str` - 線を引くアノテーションアクションを返します。(x1, y1)の[Cell](#Cell)の中心から(x2, y2)の[Cell](#Cell)の中心まで線を引きます。

- `text(x: int, y: int, message: str, fontsize: int = 16) -> str:` - draw text annotation actionを返します。タイルの上の (x, y) に、特定のメッセージとフォントサイズでテキストを書き込みます。

- `sidetext(message: str) -> str:` - draw side text annotation actionを返します。ビジュアライザーの側面に、そのターンに表示されるテキストを書き込みます。

なお、これらはすべて、アノテーションを作成したチームに応じて色分けされます（青またはオレンジ）。

### <u>GameConstants</u>

このオブジェクトには、最大ターン数やCityTilesのライトアップなど、すべてのゲームパラメータの定数が含まれます。

スターターキットに重大な変更があった場合、通常はこのオブジェクトだけが変更されます。
