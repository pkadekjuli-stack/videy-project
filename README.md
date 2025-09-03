  },
});

const upload = multer({
  storage,
  limits: {
    // up to 2 GB; adjust as needed
    fileSize: 2 * 1024 * 1024 * 1024,
  },
  fileFilter: (_, file, cb) => {
    const ext = path.extname(file.originalname).toLowerCase();
    const isVideoMime = (file.mimetype || "").startsWith("video/");
    if (isVideoMime && allowedExt.has(ext)) cb(null, true);
    else cb(new Error("File harus video dengan ekstensi yang diizinkan."));
  },
});

// routes
app.post("/upload", upload.single("video"), (req, res) => {
  const file = req.file;
  res.json({
    ok: true,
    filename: file.filename,
    size: file.size,
    url: `/uploads/${file.filename}`,
  });
});

// list uploaded files
app.get("/files", async (_req, res) => {
  const entries = await fsp.readdir(UPLOAD_DIR);
  const files = await Promise.all(
    entries.map(async (name) => {
      const full = path.join(UPLOAD_DIR, name);
      const st = await fsp.stat(full);
      return {
        name,
        size: st.size,
        createdAt: st.birthtime,
        url: `/uploads/${name}`,
      };
    })
  );
  files.sort((a, b) => b.createdAt - a.createdAt);
  res.json({ ok: true, files });
});

// simple health check
app.get("/health", (_req, res) => res.json({ ok: true }));

app.listen(PORT, () => {
  console.log(`✅ Uploader running at http://localhost:${PORT}`);
});
