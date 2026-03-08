# oh-my-claude-skills
Customized Claude Code Skills



---

Extracting the chapter contents that you want to organize from your book:

```bash
/extract-chapter <book_name> <chapter_number> <output_filename>.md

#-- Example
/extract-chapter @<book_name> chapter4 Geometry_04.md
```

---

Generating lecture notes from a video file with frame extraction and speech transcription:

```bash
/video-to-note <video-file-path> [output-filename.md]

#-- Example
/video-to-note ./lecture_07.mp4
/video-to-note ./lecture_07.mp4 LinearAlgebra_07.md
```
