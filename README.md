# =========================================================
# VIDEO XEN KẼ ULTRA ENGINE
# FINAL STABLE VERSION
# =========================================================
# FEATURES
# ✔ Chunk processing
# ✔ Static zoom
# ✔ Dynamic zoom
# ✔ Audio restore
# ✔ Preset timing
# ✔ Queue table
# ✔ Progress tracking
# ✔ Stable concat
# ✔ SAR fix
# ✔ Threading
# ✔ Multi worker
# =========================================================

import tkinter as tk
from tkinter import ttk, filedialog
import subprocess
import threading
import os
os.environ["PYTHONUTF8"] = "1"
import shutil
from datetime import datetime
from concurrent.futures import ThreadPoolExecutor

CREATE_NO_WINDOW = 0x08000000


class VideoXenKeUltra:

    def __init__(self, root):

        self.root = root

        self.root.title("VIDEO XEN KẼ ULTRA ENGINE")

        self.root.geometry("1800x1000")

        self.input_folder = tk.StringVar()
        self.output_folder = tk.StringVar()

        self.preset = tk.StringVar(value="ultrafast")

        self.crf = tk.IntVar(value=28)

        self.max_workers = tk.IntVar(value=2)

        self.ffmpeg_threads = tk.IntVar(value=4)

        self.use_custom_bitrate = tk.BooleanVar(value=True)

        self.video_bitrate = tk.StringVar(value="1000k")

        self.enable_chunk = tk.BooleanVar(value=True)

        self.chunk_threshold = tk.IntVar(value=10)

        self.chunk_size = tk.IntVar(value=10)

        self.video_list = []

        self.create_presets()

        self.build_ui()

    # =====================================================
    # PRESETS
    # =====================================================

    def create_presets(self):

        self.presets = {}

        preset_data = [

            ("10x10", 10, 10),

            ("8x6", 8, 6),

            ("6x3", 6, 3),

            ("4x2", 4, 2)

        ]

        for name, vd, sd in preset_data:

            self.presets[name] = {

                "enabled": tk.BooleanVar(value=False),

                "video_duration": tk.DoubleVar(value=vd),

                "static_duration": tk.DoubleVar(value=sd),

                "zoom": tk.DoubleVar(value=1.0),

                "static_zoom": tk.DoubleVar(value=1.10),
            }

    # =====================================================
    # UI
    # =====================================================

    def build_ui(self):

        title = tk.Label(

            self.root,

            text="VIDEO XEN KẼ ULTRA ENGINE",

            font=("Arial", 20, "bold"),

            fg="#2196F3"
        )

        title.pack(pady=10)

        # =====================================================
        # INPUT / OUTPUT
        # =====================================================

        for text, var, cmd in [

            ("Input Folder", self.input_folder, self.choose_input),

            ("Output Folder", self.output_folder, self.choose_output)

        ]:

            frame = tk.Frame(self.root)

            frame.pack(fill="x", padx=10, pady=5)

            tk.Label(
                frame,
                text=text,
                width=15,
                anchor="w"
            ).pack(side="left")

            tk.Entry(
                frame,
                textvariable=var,
                width=120
            ).pack(side="left", padx=5)

            tk.Button(
                frame,
                text="Browse",
                command=cmd
            ).pack(side="left")

        # =====================================================
        # SETTINGS
        # =====================================================

        setting_box = tk.LabelFrame(
            self.root,
            text="ENCODER SETTINGS"
        )

        setting_box.pack(fill="x", padx=10, pady=10)

        tk.Label(setting_box, text="Preset").grid(row=0, column=0)

        ttk.Combobox(

            setting_box,

            textvariable=self.preset,

            values=[
                "ultrafast",
                "superfast",
                "veryfast",
                "faster",
                "fast"
            ],

            width=15,

            state="readonly"

        ).grid(row=0, column=1)

        tk.Label(setting_box, text="CRF").grid(row=0, column=2)

        tk.Spinbox(

            setting_box,

            from_=18,
            to=35,

            textvariable=self.crf,

            width=6

        ).grid(row=0, column=3)

        tk.Label(setting_box, text="Workers").grid(row=0, column=4)

        tk.Spinbox(

            setting_box,

            from_=1,
            to=8,

            textvariable=self.max_workers,

            width=6

        ).grid(row=0, column=5)

        tk.Label(setting_box, text="Threads").grid(row=0, column=6)

        tk.Spinbox(

            setting_box,

            from_=1,
            to=64,

            textvariable=self.ffmpeg_threads,

            width=6

        ).grid(row=0, column=7)

        tk.Checkbutton(

            setting_box,

            text="Use Custom Bitrate",

            variable=self.use_custom_bitrate

        ).grid(row=1, column=0, padx=5)

        tk.Label(
            setting_box,
            text="Video Bitrate"
        ).grid(row=1, column=1)

        tk.Entry(

            setting_box,

            textvariable=self.video_bitrate,

            width=10

        ).grid(row=1, column=2)

        tk.Checkbutton(

            setting_box,

            text="Enable Chunk Processing",

            variable=self.enable_chunk

        ).grid(row=2, column=0, padx=5)

        tk.Label(
            setting_box,
            text="Chunk Threshold"
        ).grid(row=2, column=1)

        tk.Spinbox(

            setting_box,

            from_=1,
            to=120,

            textvariable=self.chunk_threshold,

            width=8

        ).grid(row=2, column=2)

        tk.Label(
            setting_box,
            text="Chunk Size"
        ).grid(row=2, column=3)

        tk.Spinbox(

            setting_box,

            from_=1,
            to=120,

            textvariable=self.chunk_size,

            width=8

        ).grid(row=2, column=4)

        # =====================================================
        # PRESETS
        # =====================================================

        preset_box = tk.LabelFrame(
            self.root,
            text="PRESETS"
        )

        preset_box.pack(fill="x", padx=10, pady=10)

        row = 0

        for name in self.presets:

            p = self.presets[name]

            tk.Checkbutton(

                preset_box,

                text=name,

                variable=p["enabled"]

            ).grid(row=row, column=0)

            tk.Label(
                preset_box,
                text="Video"
            ).grid(row=row, column=1)

            tk.Spinbox(

                preset_box,

                from_=1,
                to=60,

                textvariable=p["video_duration"],

                width=6

            ).grid(row=row, column=2)

            tk.Label(
                preset_box,
                text="Static"
            ).grid(row=row, column=3)

            tk.Spinbox(

                preset_box,

                from_=1,
                to=60,

                textvariable=p["static_duration"],

                width=6

            ).grid(row=row, column=4)

            tk.Label(
                preset_box,
                text="Zoom"
            ).grid(row=row, column=5)

            tk.Spinbox(

                preset_box,

                from_=1.0,
                to=3.0,

                increment=0.1,

                textvariable=p["zoom"],

                width=6

            ).grid(row=row, column=6)

            tk.Label(
                preset_box,
                text="Static Zoom"
            ).grid(row=row, column=7)

            tk.Spinbox(

                preset_box,

                from_=1.0,
                to=3.0,

                increment=0.05,

                textvariable=p["static_zoom"],

                width=6

            ).grid(row=row, column=8)

            row += 1

        # =====================================================
        # BUTTONS
        # =====================================================

        btn_frame = tk.Frame(self.root)

        btn_frame.pack(pady=10)

        tk.Button(

            btn_frame,

            text="LOAD VIDEO",

            bg="#2196F3",

            fg="white",

            command=self.load_videos

        ).pack(side="left", padx=5)

        tk.Button(

            btn_frame,

            text="START",

            bg="#4CAF50",

            fg="white",

            font=("Arial", 12, "bold"),

            command=self.start_processing

        ).pack(side="left", padx=5)

        # =====================================================
        # TABLE
        # =====================================================

        table_frame = tk.Frame(self.root)

        table_frame.pack(
            fill="both",
            expand=False,
            padx=10,
            pady=10
        )

        columns = (
            "video",
            "preset",
            "status",
            "progress"
        )

        self.video_table = ttk.Treeview(

            table_frame,

            columns=columns,

            show="headings",

            height=10
        )

        self.video_table.heading(
            "video",
            text="VIDEO"
        )

        self.video_table.heading(
            "preset",
            text="PRESET"
        )

        self.video_table.heading(
            "status",
            text="STATUS"
        )

        self.video_table.heading(
            "progress",
            text="PROGRESS"
        )

        self.video_table.column(
            "video",
            width=500
        )

        self.video_table.column(
            "preset",
            width=120
        )

        self.video_table.column(
            "status",
            width=180
        )

        self.video_table.column(
            "progress",
            width=300
        )

        scroll = ttk.Scrollbar(
            table_frame,
            orient="vertical",
            command=self.video_table.yview
        )

        self.video_table.configure(
            yscrollcommand=scroll.set
        )

        self.video_table.pack(
            side="left",
            fill="both",
            expand=True
        )

        scroll.pack(
            side="right",
            fill="y"
        )

        self.table_items = {}

        # =====================================================
        # LOG
        # =====================================================

        self.log_text = tk.Text(
            self.root,
            height=20,
            font=("Consolas", 9)
        )

        self.log_text.pack(
            fill="both",
            expand=True,
            padx=10,
            pady=10
        )

    # =====================================================
    # LOG
    # =====================================================

    def log(self, msg):

        ts = datetime.now().strftime("%H:%M:%S")

        self.root.after(
            0,
            lambda: self._append_log(ts, msg)
        )

    def _append_log(self, ts, msg):

        self.log_text.insert(
            tk.END,
            f"[{ts}] {msg}\n"
        )

        self.log_text.see(tk.END)

    # =====================================================
    # TABLE FUNCTIONS
    # =====================================================

    def add_table_item(

        self,

        filename,

        preset_name

    ):

        item = self.video_table.insert(

            "",

            "end",

            values=(

                filename,

                preset_name,

                "WAITING",

                "0%"
            )
        )

        self.table_items[
            (filename, preset_name)
        ] = item

    def update_table(

        self,

        filename,

        preset_name,

        status,

        progress=""
    ):

        key = (filename, preset_name)

        if key not in self.table_items:
            return

        item = self.table_items[key]

        current = self.video_table.item(
            item,
            "values"
        )

        self.video_table.item(

            item,

            values=(

                current[0],

                current[1],

                status,

                progress
            )
        )

    # =====================================================
    # CHOOSE FOLDER
    # =====================================================

    def choose_input(self):

        folder = filedialog.askdirectory()

        if folder:
            self.input_folder.set(folder)

    def choose_output(self):

        folder = filedialog.askdirectory()

        if folder:
            self.output_folder.set(folder)

    # =====================================================
    # LOAD VIDEO
    # =====================================================

    def load_videos(self):

        self.video_list.clear()

        folder = self.input_folder.get()

        if not os.path.exists(folder):

            self.log("Folder not found")

            return

        for f in os.listdir(folder):

            if f.lower().endswith((
                ".mp4",
                ".mov",
                ".avi",
                ".mkv"
            )):

                self.video_list.append(f)

        self.log(f"Loaded {len(self.video_list)} videos")

    # =====================================================
    # START
    # =====================================================

    def start_processing(self):

        threading.Thread(
            target=self.process_all,
            daemon=True
        ).start()

    # =====================================================
    # PROCESS ALL
    # =====================================================

    def process_all(self):

        tasks = []

        for file in self.video_list:

            input_path = os.path.join(
                self.input_folder.get(),
                file
            )

            for name in self.presets:

                p = self.presets[name]

                if not p["enabled"].get():
                    continue

                output_name = (
                    f"{os.path.splitext(file)[0]}"
                    f"_{name}.mp4"
                )

                output_path = os.path.join(
                    self.output_folder.get(),
                    output_name
                )

                self.root.after(

                    0,

                    lambda f=file, n=name:
                    self.add_table_item(f, n)
                )

                tasks.append((
                    file,
                    input_path,
                    output_path,
                    p
                ))

        with ThreadPoolExecutor(
            max_workers=self.max_workers.get()
        ) as executor:

            futures = []

            for task in tasks:

                futures.append(
                    executor.submit(
                        self.process_task,
                        task
                    )
                )

            for f in futures:
                f.result()

        self.log("DONE ALL")

    # =====================================================
    # VIDEO SIZE
    # =====================================================

    def get_video_size(self, path):

        result = subprocess.run([

            "ffprobe",

            "-v", "error",

            "-select_streams", "v:0",

            "-show_entries",

            "stream=width,height",

            "-of",

            "csv=s=x:p=0",

            path

        ],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            creationflags=CREATE_NO_WINDOW
        )

        output = result.stdout.decode(
            "utf-8",
            errors="ignore"
        ).strip()

        w, h = output.split("x")

        return int(w), int(h)

    # =====================================================
    # SPLIT CHUNK
    # =====================================================

    def split_chunks(self, duration):

        chunks = []

        chunk_sec = self.chunk_size.get() * 60

        start = 0

        while start < duration:

            end = min(
                start + chunk_sec,
                duration
            )

            chunks.append((start, end))

            start = end

        return chunks

    # =====================================================
    # CONCAT
    # =====================================================

    def concat_chunks(

        self,

        chunk_files,

        output_path,

        temp_dir,

        original_input

    ):

        concat_file = os.path.join(
            temp_dir,
            "concat.txt"
        )

        with open(concat_file, "w", encoding="utf-8") as f:

            for c in chunk_files:

                safe = c.replace("\\", "/")

                f.write(f"file '{safe}'\n")

        temp_video = os.path.join(
            temp_dir,
            "video_only.mp4"
        )

        cmd1 = [

            "ffmpeg",

            "-y",

            "-f", "concat",

            "-safe", "0",

            "-i", concat_file,

            "-c", "copy",

            temp_video
        ]

        subprocess.run(
            cmd1,
            creationflags=CREATE_NO_WINDOW
        )

        cmd2 = [

            "ffmpeg",

            "-y",

            "-i", temp_video,

            "-i", original_input,

            "-map", "0:v",

            "-map", "1:a?",

            "-c:v", "copy",

            "-c:a", "aac",

            "-shortest",

            output_path
        ]

        subprocess.run(
            cmd2,
            creationflags=CREATE_NO_WINDOW
        )

    # =====================================================
    # PROCESS TASK
    # =====================================================

    def process_task(self, task):

        filename, input_path, output_path, p = task

        try:

            preset_name = None

            for k, v in self.presets.items():

                if v == p:
                    preset_name = k
                    break

            self.root.after(

                0,

                lambda:
                self.update_table(

                    filename,

                    preset_name,

                    "PROCESSING",

                    "0%"
                )
            )

            self.log(f"START: {filename}")

            vw, vh = self.get_video_size(input_path)

            result = subprocess.run(
                [
                    "ffprobe",
                    "-v", "quiet",
                    "-show_entries",
                    "format=duration",
                    "-of",
                    "csv=p=0",
                    input_path
                ],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                creationflags=CREATE_NO_WINDOW
            )

            duration_text = result.stdout.decode(
                "utf-8",
                errors="ignore"
            ).strip()

            duration = float(duration_text)

            use_chunk = (

                self.enable_chunk.get()

                and

                duration >

                (self.chunk_threshold.get() * 60)
            )

            if not use_chunk:

                self.encode_segment(

                    input_path,
                    output_path,
                    0,
                    duration,
                    vw,
                    vh,
                    p
                )

            else:

                safe_preset = preset_name.replace(" ", "_")

                temp_dir = os.path.join(

                    os.getcwd(),

                    "temp_chunks",

                    f"{os.path.splitext(filename)[0]}_{safe_preset}_{threading.get_ident()}"
                )

                os.makedirs(
                    temp_dir,
                    exist_ok=True
                )

                chunk_files = []

                chunks = self.split_chunks(duration)

                for idx, (start, end) in enumerate(chunks):

                    percent = int(
                        ((idx + 1) / len(chunks)) * 100
                    )

                    self.root.after(

                        0,

                        lambda p2=percent:
                        self.update_table(

                            filename,

                            preset_name,

                            "PROCESSING",

                            f"{p2}%"
                        )
                    )

                    chunk_out = os.path.join(
                        temp_dir,
                        f"chunk_{idx}.mp4"
                    )

                    self.encode_segment(

                        input_path,
                        chunk_out,
                        start,
                        end,
                        vw,
                        vh,
                        p
                    )

                    chunk_files.append(chunk_out)

                self.concat_chunks(

                    chunk_files,

                    output_path,

                    temp_dir,

                    input_path
                )

                shutil.rmtree(
                    temp_dir,
                    ignore_errors=True
                )

            self.root.after(

                0,

                lambda:
                self.update_table(

                    filename,

                    preset_name,

                    "DONE",

                    "100%"
                )
            )

            self.log(f"DONE: {filename}")

        except Exception as e:

            self.root.after(

                0,

                lambda:
                self.update_table(

                    filename,

                    preset_name,

                    "ERROR",

                    "FAILED"
                )
            )

            self.log(str(e))

    # =====================================================
    # ENCODE
    # =====================================================

    def encode_segment(

        self,

        input_path,

        output_path,

        chunk_start,

        chunk_end,

        vw,

        vh,

        p

    ):

        filters = []

        duration = chunk_end - chunk_start

        t = 0

        seg_id = 0

        while t < duration:

            is_video = (seg_id % 2 == 0)

            seg_duration = (

                p["video_duration"].get()

                if is_video

                else p["static_duration"].get()
            )

            end_t = min(
                t + seg_duration,
                duration
            )

            zoom = (

                p["zoom"].get()

                if is_video

                else p["static_zoom"].get()
            )

            sw = int(vw * zoom)
            sh = int(vh * zoom)

            crop_x = max(
                0,
                int((sw - vw) / 2)
            )

            crop_y = max(
                0,
                int((sh - vh) / 2)
            )

            filters.append(

                f"[0:v]"
                f"trim=start={t}:end={end_t},"
                f"setpts=PTS-STARTPTS,"
                f"scale={sw}:{sh}:flags=lanczos,"
                f"crop={vw}:{vh}:{crop_x}:{crop_y},"
                f"fps=24,"
                f"format=yuv420p,"
                f"setsar=1"
                f"[v{seg_id}]"
            )

            t = end_t

            seg_id += 1

        concat_inputs = "".join(
            [f"[v{i}]" for i in range(seg_id)]
        )

        filters.append(

            f"{concat_inputs}"
            f"concat=n={seg_id}:v=1:a=0[v]"
        )

        filter_complex = ";".join(filters)

        cmd = [

            "ffmpeg",

            "-y",

            "-ss",
            str(chunk_start),

            "-to",
            str(chunk_end),

            "-i",
            input_path,

            "-filter_complex",
            filter_complex,

            "-map",
            "[v]",

            "-c:v",
            "libx264",

            "-preset",
            self.preset.get(),

            "-crf",
            str(self.crf.get()),

            "-threads",
            str(self.ffmpeg_threads.get()),

            "-pix_fmt",
            "yuv420p",

            "-r",
            "24",

            "-an"
        ]

        if self.use_custom_bitrate.get():

            bitrate_num = int(
                self.video_bitrate.get().replace("k", "")
            )

            cmd.extend([

                "-maxrate",
                self.video_bitrate.get(),

                "-bufsize",
                str(bitrate_num * 2) + "k"
            ])

        cmd.append(output_path)

        process = subprocess.Popen(
            cmd,
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
            encoding="utf-8",
            errors="ignore",
            creationflags=CREATE_NO_WINDOW
        )

        for line in process.stdout:

            line = line.strip()

            if line:

                if (
                    "frame=" in line
                    or "fps=" in line
                    or "speed=" in line
                    or "time=" in line
                ):

                    self.log(line)

        process.wait()

        if process.returncode != 0:

            self.log("FFMPEG ERROR")

        else:

            self.log(
                f"ENCODED: {os.path.basename(output_path)}"
            )


# =========================================================
# MAIN
# =========================================================

if __name__ == "__main__":

    root = tk.Tk()

    app = VideoXenKeUltra(root)

    root.mainloop()
#https://chatgpt.com/share/6a195904-4710-83ec-9942-d31e71d06dbe
