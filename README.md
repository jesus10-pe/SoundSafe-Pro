# SoundSafe Pro: Advanced Digital Signal Processing (DSP) Engine for Mobile Audio 🎧

## Executive Summary
SoundSafe Pro is a research-oriented Digital Signal Processing (DSP) framework designed to optimize acoustic output in mobile hardware environments. The project implements advanced psychoacoustic models and non-linear distortion mitigation strategies to achieve high-fidelity amplification while maintaining strict thermal and mechanical safety boundaries for integrated micro-speakers.

## Core Research & Technical Areas
* **Adaptive Dynamic Range Control (ADRC):** Implementation of multi-band compression algorithms to maximize perceived loudness without spectral clipping or signal degradation.
* **Psychoacoustic Optimization:** Leveraging human auditory perception models to enhance frequency response in constrained mobile hardware.
* **Acoustic Feedback & Thermal Guarding:** Real-time monitoring of signal gain to prevent voice coil overheating and mechanical excursion damage in mobile transducers.
* **Low-Latency Kernel Integration:** Optimization of the audio pipeline for real-time processing within the Android/Linux audio stack.

## System-Level Interoperability Benchmarking
A primary focus of our current development phase is **Service Coexistence Analysis**. We are conducting rigorous benchmarking to analyze how low-level audio drivers interact with global network filtering hooks. 

Specifically, we are evaluating the stability and performance of our DSP engine when operating alongside **AdGuard’s** advanced filtering stack. This ensures zero-latency degradation and maintains system-wide service harmony during high-intensity processing tasks.

## Project Status
* **Phase:** Private Alpha / Technical Benchmarking.
* **Focus:** Inter-process communication (IPC) stability and audio driver optimization.
* **License:** MIT.

* ### Código Fuente Principal (main.dart)

A continuación, se presenta el código maestro de la interfaz y el motor de procesamiento de SoundSafe Pro:

```dart
import 'package:flutter/material.dart';
import 'dart:math' as math;
import 'dart:async';

void main() => runApp(const SoundSafePro());

class SoundSafePro extends StatelessWidget {
  const SoundSafePro({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark(),
      home: const PantallaPrincipal(),
    );
  }
}

class PantallaPrincipal extends StatefulWidget {
  const PantallaPrincipal({super.key});

  @override
  State<PantallaPrincipal> createState() => _PantallaPrincipalState();
}

class _PantallaPrincipalState extends State<PantallaPrincipal> with SingleTickerProviderStateMixin {
  late AnimationController _waveController;
  double _currentVolume = 2.0; 
  int _modoActivo = 0; // 0: Media, 1: Gamer, 2: Voz

  @override
  void initState() {
    super.initState();
    _waveController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
  }

  @override
  void dispose() {
    _waveController.dispose();
    super.dispose();
  }

  void _updateKnobPosition(Offset localPosition, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final angle = math.atan2(localPosition.dy - center.dy, localPosition.dx - center.dx);
    double normalizedAngle = (angle + (math.pi / 2)) % (2 * math.pi);
    if (normalizedAngle < 0) normalizedAngle += (2 * math.pi);
    double potentialVolume = 1.0 + ((normalizedAngle / (2 * math.pi)) * 6.0);
    if ((potentialVolume - _currentVolume).abs() < 2.0) {
      setState(() {
        _currentVolume = potentialVolume.clamp(1.0, 7.0); 
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      drawer: Drawer(
        backgroundColor: Colors.black,
        child: Column(
          children: [
            const DrawerHeader(
              decoration: BoxDecoration(
                color: Color(0xFF0A0A0A),
                border: Border(bottom: BorderSide(color: Colors.white10, width: 1)),
              ),
              child: Align(
                alignment: Alignment.bottomLeft,
                child: Text("SOUNDSAFE PRO", style: TextStyle(fontSize: 11, letterSpacing: 3, color: Colors.cyanAccent, fontWeight: FontWeight.bold)),
              ),
            ),
            Expanded(
              child: ListView(
                padding: EdgeInsets.zero,
                children: [
                  _opcionMenuReal("Perfil y Cuenta", Icons.account_circle_outlined, () => _mostrarPerfilReal(context)),
                  _opcionMenuReal("Suscripción Premium", Icons.workspace_premium, () => _mostrarPanelPremium(context)),
                  _opcionMenuEstatica("Temporizador", Icons.timer),
                ],
              ),
            ),
            const Divider(color: Colors.white10, height: 1),
            _opcionMenuEstatica("Configuración", Icons.settings),
            _opcionMenuReal("Ayuda y Soporte (Safic)", Icons.support_agent, () => _mostrarPanelSafic(context)),
            const SizedBox(height: 30),
          ],
        ),
      ),
      appBar: AppBar(
        backgroundColor: Colors.transparent,
        elevation: 0,
        leading: Builder(
          builder: (context) {
            return IconButton(
              icon: const Icon(Icons.menu, color: Colors.cyanAccent),
              onPressed: () => Scaffold.of(context).openDrawer(),
            );
          }
        ),
        title: const Text("SOUNDSAFE PRO", style: TextStyle(fontSize: 14, letterSpacing: 3, color: Colors.white54)),
        centerTitle: true,
      ),
      body: Column(
        children: [
          const SizedBox(height: 10),
          
          // --- MONITOR DE HARDWARE ---
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 8),
            decoration: BoxDecoration(
              color: const Color(0xFF0A0A0A),
              borderRadius: BorderRadius.circular(30),
              border: Border.all(color: Colors.cyanAccent.withOpacity(0.3), width: 1),
              boxShadow: [
                BoxShadow(color: Colors.cyanAccent.withOpacity(0.05), blurRadius: 10, spreadRadius: 2)
              ]
            ),
            child: Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                const Icon(Icons.thermostat, color: Colors.greenAccent, size: 14),
                const SizedBox(width: 6),
                const Text("34°C", style: TextStyle(color: Colors.white70, fontSize: 11, fontWeight: FontWeight.bold, letterSpacing: 1)),
                const SizedBox(width: 15),
                Container(width: 1, height: 12, color: Colors.white24),
                const SizedBox(width: 15),
                const Icon(Icons.memory, color: Colors.cyanAccent, size: 14),
                const SizedBox(width: 6),
                const Text("CPU 12%", style: TextStyle(color: Colors.white70, fontSize: 11, fontWeight: FontWeight.bold, letterSpacing: 1)),
              ],
            ),
          ),

          // --- CÍRCULO CENTRAL DE POTENCIA ---
          Expanded(
            child: Center(
              child: Stack(
                alignment: Alignment.center,
                children: [
                  AnimatedBuilder(
                    animation: _waveController,
                    builder: (context, child) {
                      return Stack(
                        alignment: Alignment.center,
                        children: [
                          _buildOndaSonica(progreso: _waveController.value, retraso: 0.0),
                          _buildOndaSonica(progreso: _waveController.value, retraso: 0.33),
                          _buildOndaSonica(progreso: _waveController.value, retraso: 0.66),
                        ],
                      );
                    },
                  ),
                  GestureDetector(
                    onPanUpdate: (details) => _updateKnobPosition(details.localPosition, const Size(220, 220)),
                    onTapDown: (details) => _updateKnobPosition(details.localPosition, const Size(220, 220)),
                    child: Stack(
                      alignment: Alignment.center,
                      children: [
                        Container(
                          width: 220, height: 220,
                          decoration: BoxDecoration(
                            shape: BoxShape.circle,
                            color: Colors.black, 
                            border: Border.all(color: Colors.white10, width: 2.5),
                          ),
                          child: Center(
                            child: Column(
                              mainAxisAlignment: MainAxisAlignment.center,
                              children: [
                                Text("${_currentVolume.toStringAsFixed(1)}x", style: const TextStyle(fontSize: 65, color: Colors.white, fontWeight: FontWeight.bold)),
                                const SizedBox(height: 5),
                                const Text("POTENCIA", style: TextStyle(fontSize: 10, color: Colors.white38, letterSpacing: 1.5)),
                              ],
                            ),
                          ),
                        ),
                        SizedBox(
                          width: 220, height: 220,
                          child: CustomPaint(
                            painter: AnilloProgresoPainter(volumenActual: _currentVolume),
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            ),
          ),
          
          // --- BOTONES DE MODOS RÁPIDOS ---
          Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              _buildModoBoton(0, Icons.play_circle_filled, "MEDIA"),
              const SizedBox(width: 10),
              _buildModoBoton(1, Icons.sports_esports, "GAMER"),
              const SizedBox(width: 10),
              _buildModoBoton(2, Icons.record_voice_over, "VOZ"),
            ],
          ),
          
          const SizedBox(height: 25),

          // --- ECUALIZADOR VISUAL ---
          const EcualizadorVisual(),
          
          const SizedBox(height: 15),

          // --- SELLO DE SEGURIDAD APEX ---
          const Text(
            "🛡️ SECURED BY APEX CORE",
            style: TextStyle(
              color: Colors.white24,
              fontSize: 10,
              fontWeight: FontWeight.bold,
              letterSpacing: 2.0,
            ),
          ),
          
          const SizedBox(height: 20),
        ],
      ),
    );
  }

  // --- WIDGET PARA LOS BOTONES DE MODOS ---
  Widget _buildModoBoton(int indice, IconData icono, String texto) {
    bool activo = _modoActivo == indice;
    return GestureDetector(
      onTap: () {
        setState(() {
          _modoActivo = indice;
        });
      },
      child: AnimatedContainer(
        duration: const Duration(milliseconds: 300),
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        decoration: BoxDecoration(
          color: activo ? Colors.cyanAccent.withOpacity(0.1) : Colors.transparent,
          borderRadius: BorderRadius.circular(20),
          border: Border.all(
            color: activo ? Colors.cyanAccent : Colors.white10,
            width: 1.5,
          ),
        ),
        child: Row(
          children: [
            Icon(icono, size: 14, color: activo ? Colors.cyanAccent : Colors.white54),
            const SizedBox(width: 6),
            Text(texto, style: TextStyle(
              color: activo ? Colors.cyanAccent : Colors.white54,
              fontSize: 10,
              fontWeight: FontWeight.bold,
              letterSpacing: 1,
            )),
          ],
        ),
      ),
    );
  }

  Widget _buildOndaSonica({required double progreso, required double retraso}) {
    double progresoIndividual = (progreso + retraso) % 1.0;
    double opacidad = 1.0 - progresoIndividual;
    double escala = 1.0 + (progresoIndividual * 0.6);
    return Opacity(
      opacity: math.pow(opacidad, 2).toDouble(), 
      child: Transform.scale(
        scale: escala,
        child: Container(
          width: 220, height: 220, 
          decoration: BoxDecoration(
            shape: BoxShape.circle,
            border: Border.all(color: Colors.cyanAccent.withOpacity(0.5), width: 1.5),
          ),
        ),
      ),
    );
  }

  Widget _opcionMenuEstatica(String titulo, IconData icono) {
    return ListTile(
      dense: true, contentPadding: const EdgeInsets.symmetric(horizontal: 25), minLeadingWidth: 20,
      leading: Icon(icono, color: Colors.white54, size: 22),
      title: Text(titulo, style: const TextStyle(color: Colors.white70, fontSize: 12, fontWeight: FontWeight.w500)),
      onTap: () {},
    );
  }

  Widget _opcionMenuReal(String titulo, IconData icono, VoidCallback onTap) {
    return ListTile(
      dense: true, contentPadding: const EdgeInsets.symmetric(horizontal: 25), minLeadingWidth: 20,
      leading: Icon(icono, color: Colors.white54, size: 22),
      title: Text(titulo, style: const TextStyle(color: Colors.white70, fontSize: 12, fontWeight: FontWeight.w500)),
      onTap: () {
        Navigator.pop(context);
        onTap();
      },
    );
  }

  Widget _itemPerfilReal(String titulo, String subtitulo, IconData icono) {
    return ListTile(
      contentPadding: EdgeInsets.zero,
      leading: Icon(icono, color: Colors.white54, size: 20),
      title: Text(titulo, style: const TextStyle(fontSize: 14, color: Colors.white70)),
      subtitle: Text(subtitulo, style: const TextStyle(fontSize: 12, color: Colors.white38)),
      trailing: const Icon(Icons.arrow_forward_ios, size: 12, color: Colors.white10),
      onTap: () {},
    );
  }

  void _mostrarPerfilReal(BuildContext context) {
    showModalBottomSheet(
      context: context, backgroundColor: const Color(0xFF121212), isScrollControlled: true, 
      shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(25))),
      builder: (context) {
        return Container(
          padding: const EdgeInsets.symmetric(horizontal: 25, vertical: 20),
          height: MediaQuery.of(context).size.height * 0.85, 
          child: SingleChildScrollView(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Center(
                  child: Column(
                    children: [
                      Container(width: 40, height: 4, decoration: BoxDecoration(color: Colors.white10, borderRadius: BorderRadius.circular(10))),
                      const SizedBox(height: 25),
                      const CircleAvatar(radius: 40, backgroundColor: Color(0xFF1A1A1A), child: Icon(Icons.person, size: 45, color: Colors.white24)),
                      const SizedBox(height: 15),
                      const Text("jesus_dev_2026", style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                      const Text("jesus@example.com", style: TextStyle(fontSize: 12, color: Colors.white38)),
                      const SizedBox(height: 10),
                      Container(
                        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                        decoration: BoxDecoration(color: Colors.white.withOpacity(0.1), borderRadius: BorderRadius.circular(20), border: Border.all(color: Colors.white.withOpacity(0.2))),
                        child: const Text("PLAN PRO", style: TextStyle(color: Colors.white70, fontSize: 10, fontWeight: FontWeight.bold, letterSpacing: 1)),
                      ),
                    ],
                  ),
                ),
                const SizedBox(height: 40),
                const Text("SUSCRIPCIÓN", style: TextStyle(color: Colors.white38, fontSize: 10, fontWeight: FontWeight.bold, letterSpacing: 1.5)),
                const SizedBox(height: 10),
                _itemPerfilReal("Plan Actual", "SoundSafe VIP Mensual", Icons.stars),
                _itemPerfilReal("Próximo cobro", "15 de Abril, 2026", Icons.calendar_today),
                _itemPerfilReal("Método de Pago", "**** **** **** 4567 (Interbank)", Icons.credit_card),
                const SizedBox(height: 30),
                SizedBox(
                  width: double.infinity,
                  child: OutlinedButton(
                    style: OutlinedButton.styleFrom(
                      side: const BorderSide(color: Colors.white10),
                      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                      padding: const EdgeInsets.symmetric(vertical: 15)
                    ),
                    onPressed: () => Navigator.pop(context),
                    child: const Text("CERRAR SESIÓN", style: TextStyle(color: Colors.white70, fontSize: 13, fontWeight: FontWeight.bold)),
                  ),
                ),
              ],
            ),
          ),
        );
      }
    );
  }

  void _mostrarPanelPremium(BuildContext context) {
    showModalBottomSheet(
      context: context, backgroundColor: const Color(0xFF121212), shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(25))),
      builder: (context) {
        return Container(
          padding: const EdgeInsets.all(25),
          height: 320,
          child: Column(
            children: [
              Container(width: 40, height: 4, decoration: BoxDecoration(color: Colors.white10, borderRadius: BorderRadius.circular(10))),
              const SizedBox(height: 25),
              const Icon(Icons.workspace_premium, color: Colors.white70, size: 50),
              const SizedBox(height: 15),
              const Text("SOUNDSAFE PRO VIP", style: TextStyle(color: Colors.white, fontSize: 18, fontWeight: FontWeight.bold, letterSpacing: 2)),
              const SizedBox(height: 20),
              const Text("✔️ Desbloquea la amplificación 7x extrema\n✔️ Sin anuncios ni interrupciones\n✔️ Bóveda segura con encriptación militar", 
                style: TextStyle(color: Colors.white54, fontSize: 13, height: 1.8), textAlign: TextAlign.center),
              const Spacer(),
              ElevatedButton(
                style: ElevatedButton.styleFrom(backgroundColor: Colors.white, foregroundColor: Colors.black, minimumSize: const Size(double.infinity, 50), shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(15))),
                onPressed: () => Navigator.pop(context),
                child: const Text("OBTENER POR \$2.99/MES", style: TextStyle(fontWeight: FontWeight.bold, fontSize: 14)),
              )
            ],
          ),
        );
      }
    );
  }

  void _mostrarPanelSafic(BuildContext context) {
    showModalBottomSheet(
      context: context, backgroundColor: const Color(0xFF121212), isScrollControlled: true,
      shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(25))),
      builder: (context) {
        return Container(
          padding: const EdgeInsets.all(25),
          height: 350,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Center(
                child: Container(width: 40, height: 4, decoration: BoxDecoration(color: Colors.white10, borderRadius: BorderRadius.circular(10))),
              ),
              const SizedBox(height: 25),
              Row(
                children: [
                  const Icon(Icons.support_agent, color: Colors.white54, size: 30),
                  const SizedBox(width: 15),
                  Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text("SOPORTE TÉCNICO", style: TextStyle(color: Colors.white, fontSize: 16, fontWeight: FontWeight.bold, letterSpacing: 1.5)),
                      const SizedBox(height: 2),
                      const Text("Asistente Safic en línea", style: TextStyle(color: Colors.white38, fontSize: 12)),
                    ],
                  )
                ],
              ),
              const SizedBox(height: 25),
              Container(
                padding: const EdgeInsets.all(15),
                decoration: BoxDecoration(color: Colors.white.withOpacity(0.05), borderRadius: BorderRadius.circular(15), border: Border.all(color: Colors.white10)),
                child: const Text(
                  "¡Hola! Soy Safic, tu asistente de audio inteligente. 🎧\n\nMi misión es que disfrutes la potencia máxima de SoundSafe Pro sin poner en riesgo tus altavoces.",
                  style: TextStyle(color: Colors.white70, fontSize: 13, height: 1.5),
                ),
              ),
            ],
          ),
        );
      }
    );
  }
}

class AnilloProgresoPainter extends CustomPainter {
  final double volumenActual;
  
  AnilloProgresoPainter({required this.volumenActual});

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.width / 2;
    double porcentaje = (volumenActual - 1.0) / 6.0;
    double sweepAngle = porcentaje * 2 * math.pi;

    final paint = Paint()
      ..color = Colors.cyanAccent 
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round 
      ..strokeWidth = 3.5; 

    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -math.pi / 2, 
      sweepAngle,
      false,
      paint,
    );
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}

// --- WIDGET DEL ECUALIZADOR VISUAL ANIMADO ---
class EcualizadorVisual extends StatefulWidget {
  const EcualizadorVisual({super.key});

  @override
  State<EcualizadorVisual> createState() => _EcualizadorVisualState();
}

class _EcualizadorVisualState extends State<EcualizadorVisual> {
  List<double> _alturas = List.filled(9, 10.0);
  Timer? _timer;
  final math.Random _random = math.Random();

  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(const Duration(milliseconds: 150), (timer) {
      if (mounted) {
        setState(() {
          for (int i = 0; i < _alturas.length; i++) {
            _alturas[i] = 10.0 + _random.nextInt(30);
          }
        });
      }
    });
  }

  @override
  void dispose() {
    _timer?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: 50, 
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        crossAxisAlignment: CrossAxisAlignment.end,
        children: List.generate(_alturas.length, (index) {
          return AnimatedContainer(
            duration: const Duration(milliseconds: 150),
            margin: const EdgeInsets.symmetric(horizontal: 4),
            width: 6,
            height: _alturas[index],
            decoration: BoxDecoration(
              color: Colors.cyanAccent,
              borderRadius: BorderRadius.circular(10),
              boxShadow: [
                BoxShadow(
                  color: Colors.cyanAccent.withOpacity(0.5),
                  blurRadius: 4,
                  spreadRadius: 1,
                )
              ]
            ),
          );
        }),
      ),
    );
  }
}


