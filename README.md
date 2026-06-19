%% Simulacion del sistema cardiovascular - modelo Windkessel de 3 elementos
% Taller 1 - Laboratorio de Sistemas de Control
% Entrada: Q(t), flujo sanguineo/gasto cardiaco equivalente
% Salida: P(t), presion arterial fisiologica
% Medicion: Pm(t) = P(t) + n(t)

clear; clc; close all;

%% PARAMETROS QUE SE PUEDEN CAMBIAR
% Estos valores son referenciales. Para cambiar la situacion simulada,
% modifique Rc, Rp, C, Q_base, Q_amp, f_card y noise_percent.

% --- Caso nominal: paciente en condicion base ---
Rc = 0.05;          % resistencia caracteristica [mmHg*s/mL]
Rp = 1.00;          % resistencia periferica [mmHg*s/mL]
C  = 1.50;          % compliance arterial [mL/mmHg]
Q_base = 1.00;      % flujo medio o gasto cardiaco normalizado
Q_amp = 0.35;       % amplitud de la pulsacion del flujo
f_card = 1.20;      % frecuencia cardiaca [Hz], 1.2 Hz aprox. 72 lat/min
noise_percent = 3;  % ruido del sensor en porcentaje de la senal

% EJEMPLOS LISTOS PARA COPIAR, PEGAR O DESCOMENTAR
% Para usar un caso, quite el simbolo % de las lineas del bloque elegido.

% --- Ejemplo 1: hipertension ---
% Mayor resistencia periferica y menor elasticidad arterial.
% Rc = 0.05;
% Rp = 1.80;
% C  = 0.80;
% Q_base = 1.00;
% Q_amp = 0.35;
% f_card = 1.20;
% noise_percent = 3;

% --- Ejemplo 2: hipotension ---
% Menor resistencia periferica y vasos mas distensibles.
% Rc = 0.05;
% Rp = 0.50;
% C  = 2.00;
% Q_base = 0.85;
% Q_amp = 0.25;
% f_card = 1.10;
% noise_percent = 3;

% --- Ejemplo 3: ejercicio o aumento del gasto cardiaco ---
% Aumenta el flujo medio, la amplitud de pulsacion y la frecuencia cardiaca.
% Rc = 0.05;
% Rp = 0.90;
% C  = 1.50;
% Q_base = 1.50;
% Q_amp = 0.50;
% f_card = 1.80;    % 1.8 Hz aprox. 108 lat/min
% noise_percent = 3;

% --- Ejemplo 4: sensor con mucho ruido ---
% Mantiene el paciente nominal, pero aumenta el error de medicion.
% Rc = 0.05;
% Rp = 1.00;
% C  = 1.50;
% Q_base = 1.00;
% Q_amp = 0.35;
% f_card = 1.20;
% noise_percent = 10;

%% Funcion de transferencia P(s)/Q(s)
% Ecuacion:
% Rp*C*dP/dt + P = (Rc + Rp)*Q + Rc*Rp*C*dQ/dt
%
% En Laplace:
% P(s)/Q(s) = (Rc*Rp*C*s + Rc + Rp)/(Rp*C*s + 1)

num = [Rc*Rp*C, Rc + Rp];
den = [Rp*C, 1];
G = tf(num, den);

disp('Funcion de transferencia P(s)/Q(s):');
G

disp('Polos del sistema:');
pole(G)

disp('Ceros del sistema:');
zero(G)

%% Vector de tiempo
% IMPORTANTE: ejecutar el archivo completo con Run. Si se ejecuta solo una
% seccion, MATLAB puede no haber creado todavia la variable t.
t = 0:0.01:20;

%% Caso 1: entrada escalon en el flujo
% Representa un cambio brusco del gasto cardiaco.
Q_step = Q_base * ones(size(t));
P_step = lsim(G, Q_step, t);

figure;
plot(t, P_step, 'LineWidth', 1.8);
grid on;
xlabel('Tiempo [s]');
ylabel('Presion P(t) [mmHg]');
title('Respuesta del modelo ante entrada escalon de flujo');

%% Caso 2: entrada pulsatile
% Representa una componente periodica asociada al bombeo cardiaco.
Q_pulse = Q_base + Q_amp*sin(2*pi*f_card*t);
P_pulse = lsim(G, Q_pulse, t);

figure;
plot(t, Q_pulse, '--', 'LineWidth', 1.2); hold on;
plot(t, P_pulse, 'LineWidth', 1.8);
grid on;
xlabel('Tiempo [s]');
ylabel('Amplitud normalizada / Presion');
title('Respuesta ante flujo pulsatile');
legend('Entrada Q(t)', 'Salida P(t)', 'Location', 'best');

%% Caso 3: medicion con ruido del sensor
rng(1);
noise_sigma = (noise_percent/100) * max(abs(P_pulse));
n = noise_sigma * randn(size(t));
Pm = P_pulse(:)' + n;

figure;
plot(t, P_pulse, 'LineWidth', 1.8); hold on;
plot(t, Pm, ':', 'LineWidth', 1.0);
grid on;
xlabel('Tiempo [s]');
ylabel('Presion [mmHg]');
title('Presion fisiologica y presion medida con ruido');
legend('P(t)', 'Pm(t) = P(t) + n(t)', 'Location', 'best');

%% Caso 4: variacion de parametros
% Se modifica Rp para observar el efecto de cambios en resistencia vascular.
Rp_values = [0.6, 1.0, 1.6];

figure;
hold on;
for k = 1:length(Rp_values)
    Rp_i = Rp_values(k);
    G_i = tf([Rc*Rp_i*C, Rc + Rp_i], [Rp_i*C, 1]);
    P_i = lsim(G_i, Q_step, t);
    plot(t, P_i, 'LineWidth', 1.7, ...
        'DisplayName', ['Rp = ', num2str(Rp_i)]);
end
grid on;
xlabel('Tiempo [s]');
ylabel('Presion P(t) [mmHg]');
title('Efecto de la resistencia periferica Rp');
legend('Location', 'best');

%% Caso 5: variacion de compliance arterial
% Una menor compliance representa vasos mas rigidos.
C_values = [0.8, 1.5, 2.5];

figure;
hold on;
for k = 1:length(C_values)
    C_i = C_values(k);
    G_i = tf([Rc*Rp*C_i, Rc + Rp], [Rp*C_i, 1]);
    P_i = lsim(G_i, Q_step, t);
    plot(t, P_i, 'LineWidth', 1.7, ...
        'DisplayName', ['C = ', num2str(C_i)]);
end
grid on;
xlabel('Tiempo [s]');
ylabel('Presion P(t) [mmHg]');
title('Efecto de la compliance arterial C');
legend('Location', 'best');

%% Interpretacion rapida para el informe
% - Si Rp aumenta, la presion final aumenta porque existe mayor oposicion
%   al flujo sanguineo.
% - Si C aumenta, la respuesta se vuelve mas lenta porque el sistema tiene
%   mayor capacidad de almacenamiento/distension.
% - El polo principal esta en s = -1/(Rp*C). Como Rp y C son positivos,
%   el polo queda en el semiplano izquierdo y el sistema es estable.
% - El ruido del sensor afecta Pm(t), por lo que el dispositivo deberia
%   filtrar o procesar la medicion antes de tomar decisiones de control.
