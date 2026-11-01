# Smart GPU Task Scheduler

Routes matrix-multiplication workloads between your local CPU and a remote cloud GPU. For each task, the scheduler measures network latency and task size, then decides where to run it. A Streamlit dashboard shows the decision and compares local and cloud timings.

## How It Works

1. The scheduler pings the cloud GPU server and measures round-trip latency.
2. It applies a simple routing rule:

   | Condition | Decision |
   |---|---|
   | Matrix size < 500 | Local CPU |
   | Latency > 3.0 s | Local CPU |
   | Otherwise | Cloud GPU |

3. The dashboard runs the multiplication locally (NumPy `matmul`) and on the cloud GPU, so you can compare timings.

## Project Structure

```
GPU_task_scheduler/
├── scheduler.py    # Streamlit app: routing logic, local/cloud runners, UI
├── README.md
└── .gitattributes
```

## Requirements

- Python 3.8 or newer
- Python packages: `streamlit`, `numpy`, `requests`
- A cloud GPU server that exposes the API described below (for example, a Google Colab notebook tunneled with ngrok)

## Setup

**Step 1: Clone the repository**

```bash
git clone https://github.com/Venu277/GPU_task_scheduler.git
cd GPU_task_scheduler
```

**Step 2: Create and activate a virtual environment**

```bash
python -m venv venv
source venv/bin/activate        # macOS/Linux
venv\Scripts\activate           # Windows
```

**Step 3: Install dependencies**

```bash
pip install streamlit numpy requests
```

**Step 4: Start the cloud GPU server**

Run a server that implements the [Cloud Server API](#cloud-server-api) on a GPU runtime, and expose it publicly (for example, with ngrok). Copy the public URL it gives you.

**Step 5: Point the scheduler at your server**

Open `scheduler.py` and set `COLAB_URL` near the top of the file:

```python
COLAB_URL = "https://your-subdomain.ngrok-free.dev"
```

**Step 6: Run the app**

```bash
streamlit run scheduler.py
```

Streamlit opens the dashboard at `http://localhost:8501`.

## Cloud Server API

The scheduler expects the following endpoints. Every request sends the header `ngrok-skip-browser-warning: true`.

| Method | Endpoint | Request Body | Response |
|---|---|---|---|
| `GET` | `/ping` | None | Any successful HTTP response (used for latency measurement) |
| `POST` | `/run_task` | `{"matrix_size": <int>}` | `{"gpu_time": <float>}`, the GPU compute time in seconds |

## Usage

1. Move the **Matrix Size** slider (100 to 3000, in steps of 100).
2. Click **Run Task**.
3. Review the results:
   - **Network Latency**: measured ping time to the cloud server.
   - **Scheduler Decision**: `LOCAL` or `CLOUD`.
   - **Compute Time** (local) and **Total Time** / **Pure GPU Compute Time** (cloud).
   - A summary banner showing the speedup, or why the scheduler chose local.

## Known Limitations

- **Hardcoded server URL.** `COLAB_URL` must be updated by hand whenever the ngrok address changes.
- **Both paths always run.** Clicking Run Task executes the local and cloud computations, so the scheduler's decision affects only which result is highlighted, not whether work is done.
- **Fixed thresholds.** The size and latency cutoffs are constants in the code, not tuned from measurements.
- **Mixed timing metrics.** The cloud "Total Time" includes network overhead, but the speedup figure compares local compute against GPU-only compute.
- **No error handling.** If the server is unreachable or returns an unexpected response, the app will raise an error.
- **Cloud server not included.** The GPU server code is not part of this repository.

## License

Add a license here (for example, MIT) if you plan to share or reuse this project.
