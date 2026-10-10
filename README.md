# AlbokaSuperCollider
Alboka y pandero SeperCollider - Inar Lezaun

(
// ============================================================
// SÍNTESIS DE ALBOKA + PANDERO VASCO
// EZPATADANTZA — INAR LEZAUN
// Negra = 195 BPM
// Agrupaciones: 3,3,2,2,2...
// Microafinación, trinos y Sol5 excepcional con glissando
// Redobles progresivos y humanización rítmica
// Reverb estéreo compartida estilo SilverVerb
// INTRO: LA4 CONTINUO + DOS OSTINATOS DE PANDERO
// ============================================================

s.waitForBoot({

    // --------------------------------------------------------
    // 1. LIMPIEZA
    // --------------------------------------------------------

    if(~introRoutine.notNil) {
        ~introRoutine.stop;
        ~introRoutine = nil;
    };

    if(~albokaRoutine.notNil) {
        ~albokaRoutine.stop;
        ~albokaRoutine = nil;
    };

    if(~panderoRoutine.notNil) {
        ~panderoRoutine.stop;
        ~panderoRoutine = nil;
    };

    if(~albokaSynth.notNil) {
        ~albokaSynth.free;
        ~albokaSynth = nil;
    };

    if(~albokaReverb.notNil) {
        ~albokaReverb.free;
        ~albokaReverb = nil;
    };

    if(~albokaBus.notNil) {
        ~albokaBus.free;
        ~albokaBus = nil;
    };

    TempoClock.default.tempo = 195 / 60;

    // --------------------------------------------------------
    // 2. BUS ESTÉREO COMPARTIDO
    // --------------------------------------------------------

    ~albokaBus = Bus.audio(s, 2);

    // --------------------------------------------------------
    // 3. SÍNTESIS DE ALBOKA
    // --------------------------------------------------------

    SynthDef(\alboka, {
        |freq = 450, amp = 0.22, drone = 0,
        cents = 5, glide = 0.012, out = 0|

        var f, source, harmonics, formants;
        var reedNoise, tone, pedal, sig, breath;
        var reed, brightness;

        // Respuesta rápida para trinos articulados.
        f = Lag.kr(
            freq * (2 ** (cents / 1200)),
            glide.max(0.001)
        );

        // Modelo simplificado de lengüeta doble.
        reed = Pulse.ar(f, 0.39, 0.65)
             + Saw.ar(f, 0.42);

        // Realce no lineal de armónicos.
        source = (reed * 1.35).tanh;

        harmonics = Mix([
            SinOsc.ar(f * 2, 0, 0.12),
            SinOsc.ar(f * 3, 0, 0.11),
            SinOsc.ar(f * 4, 0, 0.10),
            SinOsc.ar(f * 5, 0, 0.085),
            SinOsc.ar(f * 6, 0, 0.065),
            SinOsc.ar(f * 7, 0, 0.060),
            SinOsc.ar(f * 8, 0, 0.045),
            SinOsc.ar(f * 9, 0, 0.035),
            SinOsc.ar(f * 11, 0, 0.025)
        ]);

        source = source + harmonics;

        // Resonancias nasales y metálicas.
        formants = Mix([
            BPF.ar(source, 880, 0.13, 1.4),
            BPF.ar(source, 1330, 0.11, 1.3),
            BPF.ar(source, 1770, 0.10, 1.1),
            BPF.ar(source, 2350, 0.10, 1.2),
            BPF.ar(source, 3500, 0.085, 1.5),
            BPF.ar(source, 4100, 0.09, 1.5),
            BPF.ar(source, 5100, 0.12, 0.8),
            BPF.ar(source, 6800, 0.16, 0.45)
        ]);

        // Ruido sutil de lengüeta.
        reedNoise = BPF.ar(
            WhiteNoise.ar(0.018),
            (f * 9).clip(1800, 9500),
            0.28
        );

        brightness = 0.85;

        tone = (source * 0.62)
             + (formants * 0.38)
             + reedNoise;

        // Pedal de La4 rico en armónicos.
        pedal = Pulse.ar(440, 0.39, 0.65)
              + Saw.ar(440, 0.42)
              + Mix([
                    SinOsc.ar(880, 0, 0.13),
                    SinOsc.ar(1320, 0, 0.11),
                    SinOsc.ar(1760, 0, 0.09),
                    SinOsc.ar(2200, 0, 0.075),
                    SinOsc.ar(2640, 0, 0.06),
                    SinOsc.ar(3080, 0, 0.05),
                    SinOsc.ar(3960, 0, 0.035),
                    SinOsc.ar(4840, 0, 0.025)
                ]);

        // Variación lenta de presión.
        breath = LFNoise2.kr(1.2).range(0.94, 1.0);

        sig = tone + (pedal * drone * 0.65);

        // Conservación de agudos y control de picos.
        sig = RLPF.ar(sig, 12500, 0.72);
        sig = LeakDC.ar(sig);
        sig = (sig * 1.35).tanh;

        // Salida al bus estéreo compartido.
        Out.ar(
            out,
            Pan2.ar(
                sig * amp * breath * brightness,
                0
            )
        );

    }).add;


    // --------------------------------------------------------
    // 4. SÍNTESIS DEL PANDERO VASCO
    // Membrana, golpe de mano y sonajas metálicas
    // --------------------------------------------------------

    SynthDef(\panderoVasco, {
        |out = 0, accent = 0.58, amp = 0.25,
        pan = 0, tone = 135, jingle = 0.8|

        var bodyEnv, metalEnv;
        var membrane, handNoise, body;
        var metalNoise, jingles, sig;
        var f, strength;

        strength = accent.clip(0.1, 1.5);

        // Variación natural del tono.
        f = tone * LFNoise2.kr(24).range(0.94, 1.06);

        // Envolvente principal.
        bodyEnv = EnvGen.kr(
            Env.perc(0.001, 0.15, 1, -7)
        );

        // Membrana grave.
        membrane = SinOsc.ar(
            f * EnvGen.kr(
                Env.perc(0.001, 0.075, 0.5, -8)
            ).range(1.65, 1.0)
        );

        // Ataque de mano.
        handNoise = BPF.ar(
            WhiteNoise.ar(1),
            1000,
            0.85
        ) * EnvGen.kr(
            Env.perc(0.001, 0.032, 1, -8)
        );

        body = (
            membrane * 0.85
            + handNoise * 0.30
        ) * bodyEnv;

        // Envolvente de sonajas y liberación automática.
        metalEnv = EnvGen.kr(
            Env.perc(0.001, 0.24, 1, -5),
            doneAction: 2
        );

        metalNoise = WhiteNoise.ar(0.7);

        // Resonancias inarmónicas de las sonajas.
        jingles = Mix([
            Ringz.ar(
                metalNoise,
                Rand(2250, 2500),
                0.055
            ),
            Ringz.ar(
                metalNoise,
                Rand(3300, 3750),
                0.065
            ),
            Ringz.ar(
                metalNoise,
                Rand(4700, 5300),
                0.050
            ),
            Ringz.ar(
                metalNoise,
                Rand(6100, 7200),
                0.040
            )
        ]) * metalEnv * jingle;

        // Cuerpo y sonajas según el acento.
        sig = (
            body * (0.65 + strength * 0.65)
            + jingles * (0.16 + strength * 0.18)
        );

        sig = HPF.ar(sig, 75);
        sig = LeakDC.ar(sig);
        sig = (sig * 1.4).tanh;

        Out.ar(
            out,
            Pan2.ar(sig * amp, pan)
        );

    }).add;


    // --------------------------------------------------------
    // 5. REVERB ESTÉREO — ESTILO SILVERVERB
    // --------------------------------------------------------

    SynthDef(\albokaReverb, {
        |inBus = 0, outBus = 0,
        mix = 0.12,
        room = 0.85,
        damp = 0.45,
        predelay = 0.008,
        amp = 1.0|

        var dry, wet, sig;

        dry = In.ar(inBus, 2);

        // Predelay de 8 ms.
        wet = DelayC.ar(
            dry,
            0.1,
            predelay.clip(0, 0.1)
        );

        // Reverberación estéreo difusa.
        wet = FreeVerb2.ar(
            wet[0],
            wet[1],
            mix: 1.0,
            room: room,
            damp: damp
        );

        wet = HPF.ar(wet, 180);
        wet = LPF.ar(wet, 8500);

        // Mezcla seca y reverberada.
        sig = (dry * (1 - mix)) + (wet * mix);

        sig = LeakDC.ar(sig);
        sig = Limiter.ar(sig, 0.95);

        Out.ar(outBus, sig * amp);

    }).add;


    s.sync;


    // --------------------------------------------------------
    // 6. ACTIVAR LA REVERB
    // --------------------------------------------------------

    ~albokaReverb = Synth(
        \albokaReverb,
        [
            \inBus, ~albokaBus.index,
            \outBus, 0,
            \mix, 0.12,
            \room, 0.85,
            \damp, 0.45,
            \predelay, 0.008,
            \amp, 1.0
        ]
    );


    // --------------------------------------------------------
    // 7. VOZ CONTINUA DE ALBOKA
    // Se crea ahora, pero la introducción la controla.
    // --------------------------------------------------------

    ~albokaSynth = Synth(
        \alboka,
        [
            \freq, 440,
            \amp, 0.22,
            \drone, 0,
            \cents, 0,
            \glide, 0.01,
            \out, ~albokaBus.index
        ]
    );


    // --------------------------------------------------------
    // 8. ESCALA DE LA DÓRICO
    // Sol5 reservado para el gesto especial
    // --------------------------------------------------------

    ~albokaScale = [69, 71, 72, 74, 76, 78];

    // Agrupaciones de corcheas.
    ~ezpatadantzaGroups = [3, 3, 2, 2, 2];


    // --------------------------------------------------------
    // 9. RITMO DEL PANDERO VASCO
    // Redobles progresivos y humanización temporal
    // La rutina se define aquí, pero aún no se reproduce.
    // --------------------------------------------------------

    ~panderoAccents = [0, 3, 6, 8, 10];

    ~panderoPatterns = [
        [2, 1, 2, 1, 2, 2, 2],
        [2, 1, 2, 1, 2, 1, 1, 2],
        [2, 1, 2, 1, 2, 2, 1, 1],
        [2, 1, 2, 1, 2, 1, 1, 1, 1]
    ];

    // Una corchea equivale a medio pulso de negra.
    ~panderoEighth = 0.5;

    ~panderoRoutine = Routine({

        var pattern;
        var position;
        var duration;
        var accent;
        var strong;
        var patternIndex = 0;
        var eighth = ~panderoEighth;
        var timing;
        var roll;
        var rollAmps;

        // Intensidad progresiva del redoble.
        rollAmps = [0.22, 0.27, 0.34, 0.43];

        inf.do({

            // Patrón básico predominante.
            if(0.25.coin) {
                patternIndex = rrand(
                    1,
                    ~panderoPatterns.size - 1
                );
            } {
                patternIndex = 0;
            };

            pattern = ~panderoPatterns[patternIndex];
            position = 0;

            pattern.do({ |units|

                duration = units * eighth;

                accent = ~panderoAccents.includes(position);
                strong = if(accent, 1.0, 0.58);

                // Redoble en la última negra cuando
                // esta ocupa dos corcheas.
                roll = (
                    (position == 10)
                    and: { units == 2 }
                    and: { 0.20.coin }
                );

                if(roll) {

                    // Cuatro golpes de intensidad creciente.
                    4.do({ |j|

                        Synth(
                            \panderoVasco,
                            [
                                \out, ~albokaBus.index,
                                \accent, rollAmps[j] * 2.0,
                                \amp, rollAmps[j],
                                \tone, rrand(122, 153),
                                \jingle, rrand(0.75, 1.0),
                                \pan, rrand(-0.10, 0.10)
                            ]
                        );

                        // Variación temporal entre golpes.
                        if(j < 3) {
                            (
                                0.25 + rrand(-0.012, 0.012)
                            ).max(0.02).wait;
                        };
                    });

                    // Completar la duración de una negra.
                    0.25.wait;

                } {

                    // Golpe normal con variación de intensidad.
                    Synth(
                        \panderoVasco,
                        [
                            \out, ~albokaBus.index,
                            \accent, strong,

                            \amp,
                            (
                                if(accent, 0.34, 0.25)
                                * rrand(0.92, 1.07)
                            ),

                            \tone, rrand(125, 150),

                            \jingle, rrand(
                                if(accent, 0.90, 0.65),
                                if(accent, 1.0, 0.90)
                            ),

                            \pan, rrand(-0.10, 0.10)
                        ]
                    );

                    // Humanización temporal del pandero.
                    timing = duration + rrand(-0.018, 0.018);
                    timing = timing.max(0.02);
                    timing.wait;
                };

                position = position + units;
            });
        });

    });


    // --------------------------------------------------------
    // 10. IMPROVISACIÓN DE ALBOKA
    // Humanización de notas, trinos y Sol5
    // La rutina se define aquí, pero aún no se reproduce.
    // --------------------------------------------------------

    ~albokaRoutine = Routine({

        var groupIndex = 0;
        var groupLength;
        var step = 0;
        var midi;
        var drone;
        var previousMidi = 69;
        var phrase;
        var accent;
        var notePool;
        var eighth = 0.5;
        var specialSol;
        var centsValue;
        var noteDuration;
        var upperMidi;
        var trill;
        var subDur;
        var trillSteps;
        var timingJitter;

        phrase = [
            69, 71, 72, 74, 72, 71,
            69, 72, 74, 76, 74, 72,
            71, 69, 71, 74, 76, 78,
            76, 74, 72, 74, 76, 78,
            78, 76, 74, 72, 71, 69
        ];

        inf.do({

            groupLength = ~ezpatadantzaGroups[groupIndex];

            groupIndex = (
                groupIndex + 1
            ) % ~ezpatadantzaGroups.size;

            groupLength.do({ |i|

                accent = (i == 0);

                // Sol5 excepcional con glissando.
                specialSol = (
                    accent
                    and: { previousMidi != 78 }
                    and: { 0.035.coin }
                );

                if(specialSol) {

                    // Fa#5: preparación del glissando.
                    ~albokaSynth.set(
                        \freq, 739.99,
                        \cents, rrand(3.0, 10.0),
                        \glide, 0.065,
                        \drone, 0,
                        \amp, 0.22
                    );

                    // Espera humanizada antes de Sol5.
                    (
                        0.08 + rrand(-0.015, 0.025)
                    ).max(0.02).wait;

                    // Sol5: gesto agudo y expresivo.
                    ~albokaSynth.set(
                        \freq, 783.99,
                        \cents, rrand(38.0, 48.0),
                        \glide, 0.22,
                        \drone, 1,
                        \amp, 0.24
                    );

                    // Permanencia humanizada en Sol5.
                    (
                        0.5 + rrand(-0.035, 0.045)
                    ).max(0.02).wait;

                    previousMidi = 79;
                    step = step + 1;

                } {

                    // Frase inicial y posterior improvisación.
                    if(step < phrase.size) {

                        midi = phrase[step];

                    } {

                        notePool = if(
                            accent,
                            [69, 71, 72, 74, 76, 78],
                            [
                                previousMidi - 2,
                                previousMidi - 1,
                                previousMidi,
                                previousMidi + 1,
                                previousMidi + 2
                            ].select({ |n|
                                ~albokaScale.includes(n)
                            })
                        );

                        notePool = notePool.select({ |n|
                            ~albokaScale.includes(n)
                        });

                        if(notePool.isEmpty) {
                            notePool = ~albokaScale;
                        };

                        midi = notePool.choose;
                    };

                    // Sol5 queda fuera de la melodía ordinaria.
                    midi = midi.clip(69, 78);

                    previousMidi = midi;

                    // Pedal con Mi5 y Fa#5.
                    drone = if(
                        [76, 78].includes(midi),
                        1,
                        0
                    );

                    // Microafinación.
                    centsValue = rrand(-2.0, 8.0);

                    if(midi == 78) {
                        centsValue = rrand(4.0, 10.0);
                    };

                    noteDuration = eighth;

                    // Vecina superior diatónica.
                    upperMidi = switch(midi,
                        69, 71,
                        71, 72,
                        72, 74,
                        74, 76,
                        76, 78,
                        78, 79,
                        79, 79,
                        69
                    );

                    // Trinos ocasionales.
                    trill = (
                        upperMidi != midi
                        and: {
                            if(accent) {
                                0.24.coin
                            } {
                                0.10.coin
                            }
                        }
                    );

                    if(trill) {

                        trillSteps = [
                            upperMidi, midi,
                            upperMidi, midi
                        ];

                        subDur = noteDuration / 4;

                        trillSteps.do({ |trillMidi|

                            ~albokaSynth.set(
                                \freq, trillMidi.midicps,
                                \cents, centsValue,
                                \glide, 0.008,
                                \drone, drone,
                                \amp, if(
                                    accent,
                                    0.24,
                                    0.19
                                )
                            );

                            // Humanización de cada nota del trino.
                            timingJitter = rrand(-0.018, 0.018);

                            (
                                subDur + timingJitter
                            ).max(0.01).wait;
                        });

                    } {

                        // Nota normal ligada a la siguiente.
                        ~albokaSynth.set(
                            \freq, midi.midicps,
                            \cents, centsValue,
                            \glide, 0.045,
                            \drone, drone,
                            \amp, if(
                                accent,
                                0.24,
                                0.19
                            )
                        );

                        // Humanización de notas normales.
                        timingJitter = rrand(-0.035, 0.035);

                        (
                            noteDuration + timingJitter
                        ).max(0.02).wait;
                    };

                    step = step + 1;

                };
            });
        });

    });


    // --------------------------------------------------------
    // 11. INTRODUCCIÓN: LA CONTINUO + PANDERO
    //
    // A) ALBOKA: LA4 NATURAL SOSTENIDO EN EL TIEMPO
    // B) ENTRA PANDERO SIN INTERRUMPIR EL LA
    // C) TRAS DOS OSTINATOS, COMIENZA LA COMPOSICIÓN
    // --------------------------------------------------------

    ~introRoutine = Routine({

        var corchea;
        var ostinato;

        // Duración de una corchea a 195 BPM, en segundos.
        corchea = 60 / 195 / 2;

        // Ostinato: 3 + 3 + 2 + 2 + 2.
        ostinato = [3, 3, 2, 2, 2];

        // A. La alboka comienza con LA4 y lo mantiene.
        ~albokaSynth.set(
            \freq, 440,
            \cents, 0,
            \glide, 0.01,
            \drone, 0,
            \amp, 0.22
        );

        // Mantener el LA durante los 3 segundos iniciales.
        3.0.wait;

        // B. Entra el pandero.
        // La alboka NO se silencia: mantiene el LA.
        // Se repite el ostinato dos veces.

        2.do({

            ostinato.do({ |unidades, i|

                Synth(
                    \panderoVasco,
                    [
                        \out, ~albokaBus.index,
                        \accent, if(i == 0, 1.0, 0.65),
                        \amp, if(i == 0, 0.34, 0.25),
                        \tone, rrand(125, 150),
                        \jingle, 0.85,
                        \pan, rrand(-0.08, 0.08)
                    ]
                );

                (unidades * corchea).wait;
            });
        });

        // C. Terminan los dos ostinatos.
        // Comienza la composición original.
        // Las dos rutinas arrancan juntas.

        ~panderoRoutine.reset;
        ~albokaRoutine.reset;

        ~panderoRoutine.play(TempoClock.default);
        ~albokaRoutine.play(TempoClock.default);

    }).play(SystemClock);

});
)
