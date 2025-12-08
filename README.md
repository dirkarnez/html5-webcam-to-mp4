let mediaRecorder;
let recordedBlobs;

// 1. Get user media and start preview
async function startRecording() {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
    // Display stream in a <video> element on the page for preview
    document.querySelector('video#preview').srcObject = stream;

    // Check if MP4 is supported, otherwise fallback might be necessary (e.g., 'video/webm')
    const options = { mimeType: 'video/mp4' };
    if (!MediaRecorder.isTypeSupported(options.mimeType)) {
        console.error(`${options.mimeType} is not Supported`);
        // Use an alternative, check supported types or use 'video/webm' which has wide support
        return; 
    }

    // 2. Initialize MediaRecorder
    recordedBlobs = [];
    try {
        mediaRecorder = new MediaRecorder(stream, options);
    } catch (err) {
        console.error('Exception while creating MediaRecorder:', err);
        return;
    }

    // 3. Gather data chunks
    mediaRecorder.ondataavailable = handleDataAvailable;
    mediaRecorder.start();
}

function handleDataAvailable(event) {
    if (event.data && event.data.size > 0) {
        recordedBlobs.push(event.data);
    }
}

// 4. Stop recording and save/download
function stopRecording() {
    mediaRecorder.stop();
    mediaRecorder.onstop = () => {
        const superBuffer = new Blob(recordedBlobs, { type: 'video/mp4' });
        const url = window.URL.createObjectURL(superBuffer);
        const a = document.createElement('a');
        a.style.display = 'none';
        a.href = url;
        a.download = 'webcam-recording.mp4'; // Specify filename with mp4 extension
        document.body.appendChild(a);
        a.click();
        setTimeout(() => {
            document.body.removeChild(a);
            window.URL.revokeObjectURL(url);
        }, 100);
    };
}
