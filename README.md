# 24BDA70363-FULL-STACK-2.1.2

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, initial-scale=1.0">

<title>Global Exception Handling</title>

<style>
body {
    font-family: Arial, sans-serif;
    background: #eef2f7;
    padding: 20px;
}

.container {
    max-width: 900px;
    margin: auto;
    background: white;
    padding: 25px;
    border-radius: 10px;
}

h2 {
    color: #1d4ed8;
}

input, select {
    width: 95%;
    padding: 12px;
    margin: 8px 0;
    border: 1px solid #ccc;
    border-radius: 5px;
}

button {
    background: #2563eb;
    color: white;
    border: none;
    padding: 12px 16px;
    margin: 6px;
    border-radius: 5px;
    cursor: pointer;
}

button:hover {
    background: #1e40af;
}

.error {
    color: #dc2626;
    font-weight: bold;
}

.success {
    color: #059669;
    font-weight: bold;
}

.output {
    background: #172033;
    color: #8fffa6;
    padding: 15px;
    min-height: 180px;
    white-space: pre-wrap;
    overflow-wrap: anywhere;
    border-radius: 6px;
}

.logs {
    background: #111827;
    color: #facc15;
    padding: 15px;
    min-height: 160px;
    white-space: pre-wrap;
    overflow-wrap: anywhere;
    border-radius: 6px;
}
</style>
</head>

<body>

<div class="container">

<h2>Global Exception Handling System</h2>
<p>Structured Logging and Correlation ID</p>

<h3>API Request Simulation</h3>

<label>Select Exception Type:</label>

<select id="exceptionType">
<option value="200">200 - Success</option>
<option value="400">400 - Bad Request</option>
<option value="404">404 - Not Found</option>
<option value="409">409 - Duplicate Resource</option>
<option value="500">500 - Internal Server Error</option>
</select>

<input type="text" id="resource"
placeholder="Enter Resource Name">

<input type="text" id="correlation"
placeholder="Optional Correlation ID">

<br>

<button onclick="sendRequest()">
Send API Request
</button>

<button onclick="clearOutput()">
Clear Output
</button>

<h3>API Response</h3>

<p id="statusMessage"></p>

<div class="output" id="output">
API response will appear here.
</div>

<h3>Structured Server Logs</h3>

<div class="logs" id="logs">
Waiting for API request...
</div>

</div>

<script>

function generateCorrelationId() {
    if (window.crypto && crypto.randomUUID) {
        return crypto.randomUUID();
    }

    return "req-" + Date.now() + "-" +
        Math.random().toString(36).slice(2, 10);
}

function addLog(level, correlationId, message) {

    const time = new Date().toISOString();

    const logMessage =
        "[" + time + "] " +
        "[" + level + "] " +
        "[" + correlationId + "] " +
        message + "\n";

    const logs = document.getElementById("logs");

    if (logs.textContent ===
        "Waiting for API request...") {
        logs.textContent = "";
    }

    logs.textContent += logMessage;
}

function createErrorResponse(
    status, error, message, path, correlationId
) {
    return {
        success: false,
        status: status,
        error: error,
        message: message,
        path: path,
        correlationId: correlationId,
        timestamp: new Date().toISOString()
    };
}

function globalExceptionHandler(status, resource, correlationId) {

    const path = "/api/v1/posts/" +
        encodeURIComponent(resource);

    switch (status) {

        case 400:
            return createErrorResponse(
                400,
                "Bad Request",
                "Validation failed: Invalid request data",
                path,
                correlationId
            );

        case 404:
            return createErrorResponse(
                404,
                "Not Found",
                "Resource not found: " + resource,
                path,
                correlationId
            );

        case 409:
            return createErrorResponse(
                409,
                "Conflict",
                "Duplicate resource already exists",
                path,
                correlationId
            );

        case 500:
            return createErrorResponse(
                500,
                "Internal Server Error",
                "An unexpected internal error occurred.",
                path,
                correlationId
            );

        default:
            return {
                success: true,
                status: 200,
                message: "Request processed successfully",
                data: {
                    resource: resource,
                    status: "ACTIVE"
                },
                correlationId: correlationId,
                timestamp: new Date().toISOString()
            };
    }
}

function sendRequest() {

    const startTime = performance.now();

    const status = Number(
        document.getElementById("exceptionType").value
    );

    const resource =
        document.getElementById("resource").value.trim()
        || "sample-post";

    const customId =
        document.getElementById("correlation").value.trim();

    const correlationId =
        customId || generateCorrelationId();

    const path = "/api/v1/posts/" +
        encodeURIComponent(resource);

    addLog(
        "INFO",
        correlationId,
        "Incoming GET request: " + path
    );

    const response = globalExceptionHandler(
        status, resource, correlationId
    );

    if (status >= 400) {

        addLog(
            status === 500 ? "ERROR" : "WARN",
            correlationId,
            "Exception handled: HTTP " + status
        );

    } else {

        addLog(
            "INFO",
            correlationId,
            "Request completed successfully"
        );
    }

    const duration =
        (performance.now() - startTime).toFixed(2);

    addLog(
        "INFO",
        correlationId,
        "Completed request | Status: " +
        status + " | Duration: " +
        duration + "ms"
    );

    document.getElementById("output").textContent =
        JSON.stringify(response, null, 4);

    const message =
        document.getElementById("statusMessage");

    message.textContent =
        "HTTP Status: " + status +
        " | Correlation ID: " + correlationId;

    message.className =
        status >= 400 ? "error" : "success";
}

function clearOutput() {

    document.getElementById("output").textContent =
        "API response will appear here.";

    document.getElementById("logs").textContent =
        "Waiting for API request...";

    document.getElementById("statusMessage").textContent = "";
}

</script>

</body>
</html>
