import os
import glob
import time

# Carrega os módulos do kernel (pode rodar antes também via terminal)
os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')

# Diretório base dos sensores 1-Wire
base_dir = '/sys/bus/w1/devices/'

# Encontra a pasta do sensor (começa com 28- normalmente)
device_folder = glob.glob(base_dir + '28*')

if not device_folder:
    print("Nenhum sensor DS18B20 encontrado! Verifique conexões e módulos carregados.")
    exit(1)

# Pega o primeiro sensor encontrado
device_file = device_folder[0] + '/w1_slave'

def read_temp_raw():
    """Lê as linhas brutas do sensor"""
    with open(device_file, 'r') as f:
        lines = f.readlines()
    return lines

def read_temp():
    """Lê a temperatura e verifica CRC"""
    lines = read_temp_raw()
    
    # Tenta até conseguir leitura válida (CRC = YES)
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw()
    
    # Encontra a parte com a temperatura
    equals_pos = lines[1].find('t=')
    if equals_pos == -1:
        return None  # Erro
    
    temp_string = lines[1][equals_pos + 2:]
    temp_c = float(temp_string) / 1000.0
    temp_f = temp_c * 9.0 / 5.0 + 32.0
    
    return temp_c, temp_f

print("Monitor de Temperatura - Bateria SLA (DS18B20)")
print("Ctrl+C para parar\n")

try:
    while True:
        temp_c, temp_f = read_temp()
        if temp_c is not None:
            print(f"Temperatura da bateria: {temp_c:.1f} °C  |  {temp_f:.1f} °F")
        else:
            print("Erro ao ler o sensor!")
        
        time.sleep(2)  # Lê a cada 2 segundos

except KeyboardInterrupt:
    print("\nPrograma encerrado pelo usuário.")
