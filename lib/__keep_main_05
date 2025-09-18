import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart' show rootBundle;
import 'package:flutter_map/flutter_map.dart';
import 'package:latlong2/latlong.dart';

const String kTokyoMunicipalAsset = 'assets/tokyo_municipal.geojson';

class MunicipalGeom {
  final String name;
  final String kind;
  final List<List<List<List<double>>>> polygons;

  late final double centroidLat;
  late final double centroidLng;
  int? zKey;

  MunicipalGeom({required this.name, required this.kind, required this.polygons}) {
    final c = _computeCentroid(polygons);
    centroidLat = c.$1;
    centroidLng = c.$2;
  }
}

(double, double) _computeCentroid(List<List<List<List<double>>>> polys) {
  double sumLat = 0, sumLng = 0;
  int cnt = 0;
  for (final rings in polys) {
    if (rings.isEmpty) continue;
    for (final p in rings.first) {
      sumLng += p[0];
      sumLat += p[1];
      cnt++;
    }
  }
  if (cnt == 0) return (0, 0);
  return (sumLat / cnt, sumLng / cnt);
}

Future<List<MunicipalGeom>> loadTokyoMunicipals(String assetPath) async {
  final text = await rootBundle.loadString(assetPath);
  final data = jsonDecode(text);

  final List features;
  if (data['type'] == 'FeatureCollection') {
    features = (data['features'] as List);
  } else if (data['type'] == 'Feature') {
    features = [data];
  } else {
    throw Exception('Unsupported GeoJSON root: ${data['type']}');
  }

  final list = <MunicipalGeom>[];
  for (final f in features) {
    final props = Map<String, dynamic>.from(f['properties'] ?? {});
    final geom = Map<String, dynamic>.from(f['geometry'] ?? {});
    if (geom.isEmpty) continue;

    final rawName = (props['N03_004'] ?? props['name'] ?? '') as String;
    if (rawName.isEmpty) continue;

    final kind = rawName.endsWith('区')
        ? '区'
        : rawName.endsWith('市')
        ? '市'
        : '町村';
    final type = geom['type'] as String;
    final coords = geom['coordinates'];

    if (type == 'Polygon') {
      final poly = (coords as List)
          .map<List<List<double>>>(
            (ring) => (ring as List)
                .map<List<double>>((pt) => (pt as List).map((n) => (n as num).toDouble()).toList())
                .toList(),
          )
          .toList();
      list.add(MunicipalGeom(name: rawName, kind: kind, polygons: [poly]));
    } else if (type == 'MultiPolygon') {
      final mpoly = (coords as List)
          .map<List<List<List<double>>>>(
            (poly) => (poly as List)
                .map<List<List<double>>>(
                  (ring) => (ring as List)
                      .map<List<double>>((pt) => (pt as List).map((n) => (n as num).toDouble()).toList())
                      .toList(),
                )
                .toList(),
          )
          .toList();
      list.add(MunicipalGeom(name: rawName, kind: kind, polygons: mpoly));
    }
  }
  return list;
}

class Station {
  final String name;
  final LatLng latlng;

  const Station(this.name, this.latlng);
}

const List<Station> yamanoteStations = [
  Station('東京', LatLng(35.681236, 139.767125)),
  Station('神田', LatLng(35.691731, 139.770883)),
  Station('秋葉原', LatLng(35.698353, 139.773114)),
  Station('御徒町', LatLng(35.707155, 139.774473)),
  Station('上野', LatLng(35.713768, 139.777254)),
  Station('鶯谷', LatLng(35.720495, 139.778837)),
  Station('日暮里', LatLng(35.727772, 139.770987)),
  Station('西日暮里', LatLng(35.732247, 139.766831)),
  Station('田端', LatLng(35.738178, 139.759640)),
  Station('駒込', LatLng(35.736202, 139.746837)),
  Station('巣鴨', LatLng(35.733492, 139.739345)),
  Station('大塚', LatLng(35.731332, 139.729523)),
  Station('池袋', LatLng(35.728926, 139.710380)),
  Station('目白', LatLng(35.721188, 139.706553)),
  Station('高田馬場', LatLng(35.712285, 139.703782)),
  Station('新大久保', LatLng(35.701306, 139.700043)),
  Station('新宿', LatLng(35.690921, 139.700258)),
  Station('代々木', LatLng(35.683061, 139.702042)),
  Station('原宿', LatLng(35.670302, 139.702042)),
  Station('渋谷', LatLng(35.658034, 139.701636)),
  Station('恵比寿', LatLng(35.646672, 139.710106)),
  Station('目黒', LatLng(35.633998, 139.715828)),
  Station('五反田', LatLng(35.625826, 139.723106)),
  Station('大崎', LatLng(35.619700, 139.728553)),
  Station('品川', LatLng(35.628472, 139.738760)),
  Station('高輪ゲートウェイ', LatLng(35.636520, 139.740860)),
  Station('田町', LatLng(35.645736, 139.747574)),
  Station('浜松町', LatLng(35.655391, 139.757104)),
  Station('新橋', LatLng(35.666195, 139.758601)),
  Station('有楽町', LatLng(35.675069, 139.763328)),
];

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final municipals = await loadTokyoMunicipals(kTokyoMunicipalAsset);
  final gb = _globalBoundsForCenters(municipals);
  _assignZOrderKeys(municipals, gb);
  runApp(TokyoSelectApp(municipals: municipals));
}

class TokyoSelectApp extends StatelessWidget {
  const TokyoSelectApp({super.key, required this.municipals});

  final List<MunicipalGeom> municipals;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Tokyo Select Overlay (Stations)',
      theme: ThemeData(useMaterial3: true, colorSchemeSeed: const Color(0xFFE53935)),
      home: TokyoSelectPage(municipals: municipals),
    );
  }
}

class TokyoSelectPage extends StatefulWidget {
  const TokyoSelectPage({super.key, required this.municipals});

  final List<MunicipalGeom> municipals;

  @override
  State<TokyoSelectPage> createState() => _TokyoSelectPageState();
}

class _TokyoSelectPageState extends State<TokyoSelectPage> {
  final MapController _mapController = MapController();
  String _category = '区';
  String? _selectedName;
  final _center = const LatLng(35.6895, 139.6917);

  late final Map<String, LatLngBounds> _boundsByName = {for (final m in widget.municipals) m.name: _computeBounds(m)};

  @override
  Widget build(BuildContext context) {
    final filtered = widget.municipals.where((m) => m.kind == _category).toList()
      ..sort((a, b) {
        final ka = a.zKey ?? 0, kb = b.zKey ?? 0;
        if (ka != kb) return ka.compareTo(kb);
        return a.name.compareTo(b.name);
      });

    // 区市町村ポリゴン
    final polygons = <Polygon>[];
    for (final m in widget.municipals) {
      final isSelected = (m.name == _selectedName);
      final fill = isSelected ? const Color(0x55FF0000) : const Color(0x22000000);
      final stroke = isSelected ? const Color(0xFFFF0000) : const Color(0x55000000);
      final strokeWidth = isSelected ? 2.0 : 1.0;

      for (final rings in m.polygons) {
        final outer = rings.first.map((p) => LatLng(p[1], p[0])).toList();
        final holes = <List<LatLng>>[];
        for (int i = 1; i < rings.length; i++) {
          holes.add(rings[i].map((p) => LatLng(p[1], p[0])).toList());
        }
        polygons.add(
          Polygon(
            points: outer,
            holePointsList: holes.isEmpty ? null : holes,
            isFilled: true,
            color: fill,
            borderColor: stroke,
            borderStrokeWidth: strokeWidth,
          ),
        );
      }
    }

    // 選択区内の駅だけマーカー
    final markers = <Marker>[];
    final selectedMunicipal = _selectedName == null
        ? null
        : widget.municipals.firstWhere((m) => m.name == _selectedName, orElse: () => widget.municipals.first);

    if (selectedMunicipal != null && _selectedName != null) {
      for (final st in yamanoteStations) {
        if (_isPointInsideMunicipal(st.latlng, selectedMunicipal)) {
          markers.add(
            Marker(
              width: 160,
              height: 84,
              alignment: Alignment.bottomCenter,
              // ← 地理座標＝下端中央に一致
              point: st.latlng,
              child: _StationPin(name: st.name),
            ),
          );
        }
      }
    }

    return Scaffold(
      appBar: AppBar(title: const Text('東京都 区市町村＋山手線駅マーカー')),
      body: Column(
        children: [
          // カテゴリ切替
          Padding(
            padding: const EdgeInsets.fromLTRB(12, 12, 12, 6),
            child: Wrap(
              spacing: 8,
              children: [
                _CatButton(
                  label: '23区',
                  selected: _category == '区',
                  onTap: () => setState(() {
                    _category = '区';
                    _selectedName = null;
                  }),
                ),
                _CatButton(
                  label: '26市',
                  selected: _category == '市',
                  onTap: () => setState(() {
                    _category = '市';
                    _selectedName = null;
                  }),
                ),
                _CatButton(
                  label: '町村',
                  selected: _category == '町村',
                  onTap: () => setState(() {
                    _category = '町村';
                    _selectedName = null;
                  }),
                ),
              ],
            ),
          ),
          // 近い順の名前チップ
          SizedBox(
            height: 56,
            child: ListView.separated(
              padding: const EdgeInsets.symmetric(horizontal: 12),
              scrollDirection: Axis.horizontal,
              itemCount: filtered.length,
              separatorBuilder: (_, __) => const SizedBox(width: 8),
              itemBuilder: (context, i) {
                final name = filtered[i].name;
                final selected = name == _selectedName;
                return ChoiceChip(selected: selected, label: Text(name), onSelected: (_) => _onSelectName(name));
              },
            ),
          ),
          const SizedBox(height: 8),
          // 地図
          Expanded(
            child: FlutterMap(
              mapController: _mapController,
              options: MapOptions(
                initialCenter: _center,
                initialZoom: 9.5,
                onTap: (_, __) => setState(() => _selectedName = null),
              ),
              children: [
                TileLayer(
                  urlTemplate: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
                  userAgentPackageName: 'com.example.tokyo_select_overlay',
                ),
                PolygonLayer(polygons: polygons),
                if (markers.isNotEmpty) MarkerLayer(markers: markers),
              ],
            ),
          ),
        ],
      ),
    );
  }

  void _onSelectName(String name) {
    setState(() => _selectedName = name);
    final b = _boundsByName[name];
    if (b == null) return;
    final fit = CameraFit.bounds(bounds: b, padding: const EdgeInsets.all(24));
    _mapController.fitCamera(fit);
  }

  LatLngBounds _computeBounds(MunicipalGeom m) {
    double? minLat, minLng, maxLat, maxLng;
    for (final rings in m.polygons) {
      for (final pt in rings.first) {
        final lng = pt[0], lat = pt[1];
        minLat = (minLat == null) ? lat : (lat < minLat ? lat : minLat);
        maxLat = (maxLat == null) ? lat : (lat > maxLat ? lat : maxLat);
        minLng = (minLng == null) ? lng : (lng < minLng ? lng : minLng);
        maxLng = (maxLng == null) ? lng : (lng > maxLng ? lng : maxLng);
      }
    }
    final sw = LatLng(minLat ?? 0, minLng ?? 0);
    final ne = LatLng(maxLat ?? 0, maxLng ?? 0);
    return LatLngBounds(sw, ne);
  }

  bool _isPointInsideMunicipal(LatLng p, MunicipalGeom m) {
    for (final rings in m.polygons) {
      if (_pointInPolygon(p.latitude, p.longitude, rings)) return true;
    }
    return false;
  }

  bool _pointInPolygon(double lat, double lng, List<List<List<double>>> polygon) {
    if (polygon.isEmpty) return false;
    final outer = polygon.first;
    if (!_pointInRing(lat, lng, outer)) return false;
    for (int i = 1; i < polygon.length; i++) {
      if (_pointInRing(lat, lng, polygon[i])) return false;
    }
    return true;
  }

  bool _pointInRing(double lat, double lng, List<List<double>> ring) {
    int cnt = 0;
    final int n = ring.length;
    for (int i = 0, j = n - 1; i < n; j = i++) {
      final xi = ring[i][0], yi = ring[i][1];
      final xj = ring[j][0], yj = ring[j][1];
      final intersect =
          ((yi > lat) != (yj > lat)) && (lng < (xj - xi) * (lat - yi) / ((yj - yi) == 0 ? 1e-12 : (yj - yi)) + xi);
      if (intersect) cnt++;
    }
    return (cnt % 2 == 1);
  }
}

class _GlobalBounds {
  final double minLat, maxLat, minLng, maxLng;

  const _GlobalBounds(this.minLat, this.maxLat, this.minLng, this.maxLng);
}

_GlobalBounds _globalBoundsForCenters(List<MunicipalGeom> list) {
  double minLat = 90, maxLat = -90, minLng = 180, maxLng = -180;
  for (final m in list) {
    if (m.centroidLat < minLat) minLat = m.centroidLat;
    if (m.centroidLat > maxLat) maxLat = m.centroidLat;
    if (m.centroidLng < minLng) minLng = m.centroidLng;
    if (m.centroidLng > maxLng) maxLng = m.centroidLng;
  }
  return _GlobalBounds(minLat, maxLat, minLng, maxLng);
}

void _assignZOrderKeys(List<MunicipalGeom> list, _GlobalBounds gb) {
  for (final m in list) {
    final nx = _normalize(m.centroidLng, gb.minLng, gb.maxLng);
    final ny = _normalize(m.centroidLat, gb.minLat, gb.maxLat);
    m.zKey = _mortonKey16(nx, ny);
  }
}

double _normalize(double v, double vmin, double vmax) {
  final d = (vmax - vmin);
  if (d == 0) return 0.5;
  return (v - vmin) / d;
}

int _mortonKey16(double normX, double normY) {
  int x = (normX.clamp(0.0, 1.0) * 65535).round();
  int y = (normY.clamp(0.0, 1.0) * 65535).round();
  return _interleave16(x) | (_interleave16(y) << 1);
}

int _interleave16(int n) {
  int x = n & 0xFFFF;
  x = (x | (x << 8)) & 0x00FF00FF;
  x = (x | (x << 4)) & 0x0F0F0F0F;
  x = (x | (x << 2)) & 0x33333333;
  x = (x | (x << 1)) & 0x55555555;
  return x;
}

class _CatButton extends StatelessWidget {
  const _CatButton({required this.label, required this.selected, required this.onTap});

  final String label;
  final bool selected;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    final color = selected ? Theme.of(context).colorScheme.primary : Colors.black54;
    return OutlinedButton(
      onPressed: onTap,
      style: OutlinedButton.styleFrom(
        side: BorderSide(color: color),
        foregroundColor: color,
        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
      ),
      child: Text(label),
    );
  }
}

/// 駅ピン：ラベルが“上”、アイコンが“下”（アイコン先端＝下端中央が地点に一致）
class _StationPin extends StatelessWidget {
  const _StationPin({required this.name});

  final String name;

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        // ラベル（上）
        Container(
          padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 2),
          margin: const EdgeInsets.only(bottom: 2),
          constraints: const BoxConstraints(maxWidth: 140),
          decoration: BoxDecoration(color: Colors.white.withOpacity(0.95), borderRadius: BorderRadius.circular(4)),
          child: Text(name, style: const TextStyle(fontSize: 12), softWrap: false, overflow: TextOverflow.ellipsis),
        ),
        // ピンアイコン（下端の先っぽが地点）
        const Icon(Icons.location_on, size: 28, color: Colors.red),
      ],
    );
  }
}
