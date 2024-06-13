#1
import os
import subprocess
from tkinter import filedialog, messagebox, Tk, Button, Label
import threading
from PIL import Image

#2
def eliminar_informacion_imagen(ruta_imagen):
    try:
        imagen = Image.open(ruta_imagen)
        pixeles = imagen.load()
        anchura, altura = imagen.size

        for x in range(anchura):
            for y in range(altura):
                r, g, b = pixeles[x, y]
                pixeles[x, y] = (r & ~1, g & ~1, b & ~1)
        
        nueva_ruta_imagen = f"limpia_{os.path.basename(ruta_imagen)}"
        imagen.save(nueva_ruta_imagen)
        messagebox.showinfo("Éxito", f"Contenido oculto eliminado y guardado como '{nueva_ruta_imagen}'")
    except Exception as e:
        messagebox.showerror("Error", f"Ocurrió un error al limpiar la imagen:\n{e}")

#3
def eliminar_contenido_oculto(ruta_archivo):
    try:
        extension = os.path.splitext(ruta_archivo)[1].lower()
        if extension not in ['.pdf', '.mp4', '.mp3', '.txt']:
            messagebox.showerror("Error", "Tipo de archivo no soportado. Por favor seleccione un archivo PDF, MP4, MP3 o TXT.")
            return

        temp_dir = "temp_extracted"
        if not os.path.exists(temp_dir):
            os.makedirs(temp_dir)

        cmd = f"binwalk --extract --directory={temp_dir} {ruta_archivo}"
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        print(result.stdout)
        print(result.stderr)

        with open(ruta_archivo, 'rb') as archivo:
            contenido_original = archivo.read()

        contenido_a_guardar = bytearray()
        offset_anterior = 0

        for linea in result.stdout.splitlines():
            if "DECIMAL" in linea:
                continue

            partes = linea.split()
            if len(partes) < 2:
                continue

            offset_actual = int(partes[0])
            contenido_a_guardar.extend(contenido_original[offset_anterior:offset_actual])
            offset_anterior = offset_actual + len(partes[-1].encode('utf-8'))

        contenido_a_guardar.extend(contenido_original[offset_anterior:])

        ruta_archivo_limpio = ruta_archivo.replace(".", "_limpio.")
        with open(ruta_archivo_limpio, 'wb') as archivo:
            archivo.write(contenido_a_guardar)

        for root, dirs, files in os.walk(temp_dir, topdown=False):
            for file in files:
                os.remove(os.path.join(root, file))
            for dir in dirs:
                os.rmdir(os.path.join(root, dir))
        os.rmdir(temp_dir)

        messagebox.showinfo("Éxito", f"Contenido oculto eliminado. Archivo limpio guardado como {ruta_archivo_limpio}")
    except Exception as e:
        messagebox.showerror("Error", f"Ocurrió un error al eliminar el contenido oculto:\n{e}")
#4
def seleccionar_archivo():
    ruta_archivo = filedialog.askopenfilename()
    return ruta_archivo

def iniciar_eliminacion_archivo():
    ruta_archivo = seleccionar_archivo()
    if ruta_archivo:
        thread = threading.Thread(target=eliminar_contenido_oculto, args=(ruta_archivo,))
        thread.start()

def iniciar_eliminacion_imagen():
    ruta_imagen = seleccionar_archivo()
    if ruta_imagen:
        thread = threading.Thread(target=eliminar_informacion_imagen, args=(ruta_imagen,))
        thread.start()
#5
def mostrar_menu():
    root = Tk()
    root.title("Eliminador de Contenido Oculto")

    label = Label(root, text="Seleccione una opción:")
    label.pack(pady=10)

    boton_eliminar_archivo = Button(root, text="Eliminar contenido oculto en archivo", command=iniciar_eliminacion_archivo)
    boton_eliminar_archivo.pack(pady=5)

    boton_eliminar_imagen = Button(root, text="Eliminar contenido oculto en imagen", command=iniciar_eliminacion_imagen)
    boton_eliminar_imagen.pack(pady=5)
    root.mainloop()

if __name__ == "__main__":
    mostrar_menu()
