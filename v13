(function() {
    if (document.getElementById('spx-grabber-ui')) document.getElementById('spx-grabber-ui').remove();

    let stopRequested = false;

    const div = document.createElement('div');
    div.id = 'spx-grabber-ui';
    div.style = "position:fixed;top:10px;right:10px;z-index:9999;background:#fff;padding:15px;border:2px solid #ee4d2d;border-radius:8px;box-shadow:0 4px 12px rgba(0,0,0,0.3);width:280px;font-family:Arial, sans-serif;";
    div.innerHTML = `
        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:10px;">
            <h3 style="margin:0;color:#ee4d2d;font-size:16px;">SPX Grabber v14 (Fixed SKU)</h3>
            <button id="grabber-close" style="background:#ddd; border:none; border-radius:50%; width:24px; height:24px; cursor:pointer;">×</button>
        </div>
        <textarea id="grabber-input" placeholder="Paste Tracking Numbers..." style="width:100%;height:80px;margin-bottom:10px;font-size:12px;border:1px solid #ddd;padding:5px;box-sizing:border-box;"></textarea>
        <button id="grabber-start" style="background:#ee4d2d;color:#fff;border:none;padding:10px;width:100%;cursor:pointer;font-weight:bold;border-radius:4px;">START EXTRACTION</button>
        <button id="grabber-stop" style="display:none; background:#555;color:#fff;border:none;padding:10px;width:100%;cursor:pointer;font-weight:bold;border-radius:4px;margin-top:5px;">STOP & SAVE</button>
        <div id="grabber-status" style="font-size:12px;margin-top:10px;color:#333;font-weight:bold;">Status: Ready</div>
    `;
    document.body.appendChild(div);

    const startBtn = document.getElementById('grabber-start');
    const stopBtn = document.getElementById('grabber-stop');
    const status = document.getElementById('grabber-status');

    document.getElementById('grabber-close').onclick = () => div.remove();
    stopBtn.onclick = () => { stopRequested = true; status.innerText = "Stopping..."; };

    const waitForData = async (container, labelName, i, total) => {
        if (!container) return "N/A";
        const dataBox = container.querySelector('.sensitive-wrap-data');
        
        for (let retry = 0; retry < 15; retry++) {
            let val = dataBox?.innerText.trim() || "";
            let title = dataBox?.getAttribute('title') || "";

            // Prioritize title if it exists and isn't masked
            if (title && !title.includes("*") && title.length > 5) return title.replace(/,/g, " ");
            if (val !== "" && !val.includes("*")) return val.replace(/,/g, " ");

            status.innerHTML = `[${i+1}/${total}] <span style="color:orange;">Unmasking ${labelName}...</span>`;
            const eye = container.querySelector('.sensitive-wrap-icon');
            if (eye) eye.dispatchEvent(new MouseEvent('click', {bubbles: true}));
            
            await new Promise(r => setTimeout(r, 800));
        }
        return dataBox?.innerText.trim().replace(/,/g, " ") || "N/A";
    };

    startBtn.onclick = async () => {
        const input = document.getElementById('grabber-input').value;
        const codes = input.split('\n').map(c => c.trim()).filter(c => c !== "");
        if (codes.length === 0) return alert("Paste numbers!");

        stopRequested = false;
        startBtn.style.display = 'none';
        stopBtn.style.display = 'block';
        
        let csvRows = [["Tracking Number", "Receiver Name", "Town", "Address", "SKU Name", "Qty", "Weight", "H", "W", "L"]];

        for (let i = 0; i < codes.length; i++) {
            if (stopRequested) break;
            const code = codes[i];
            window.location.hash = `#/orderDetail/${code}/sender_info`;

            // Wait for load
            let ready = false;
            for(let p=0; p<20; p++) {
                if(document.querySelector('div[data-for="buyer_name"]')) { ready = true; break; }
                await new Promise(r => setTimeout(r, 500));
            }

            // Extract Receiver Info
            const name = await waitForData(document.querySelector('div[data-for="buyer_name"]'), "Name", i, codes.length);
            const town = await waitForData(document.querySelector('div[data-for="buyer_addr_town"]'), "Town", i, codes.length);
            const addr = await waitForData(document.querySelector('div[data-for="buyer_addr"]'), "Addr", i, codes.length);

            // Extract Table Data
            const rows = document.querySelectorAll('.ssc-table-row');
            if (rows.length === 0) {
                csvRows.push([code, name, town, addr, "No SKU Found", "", "", "", "", ""]);
            } else {
                for (let tr of rows) {
                    const cells = tr.querySelectorAll('td');
                    if (cells.length >= 9) {
                        // Cells[2] is usually the SKU Name cell
                        const skuVal = await waitForData(cells[2], "SKU Name", i, codes.length);
                        
                        csvRows.push([
                            code, name, town, addr, skuVal, 
                            cells[4].innerText.trim(), 
                            cells[5].innerText.trim(), 
                            cells[6].innerText.trim(), 
                            cells[7].innerText.trim(), 
                            cells[8].innerText.trim()
                        ]);
                    }
                }
            }
        }

        const csvContent = csvRows.map(e => e.join(",")).join("\n");
        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement("a");
        link.href = URL.createObjectURL(blob);
        link.download = `SPX_Export_v14.csv`;
        link.click();

        startBtn.style.display = 'block';
        stopBtn.style.display = 'none';
        status.innerText = "Done!";
    };
})();