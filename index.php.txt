<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $projectDetails = $_POST['project_details'] ?? '';
    
    if (empty($projectDetails)) {
        die("الرجاء إدخال تفاصيل المشروع.");
    }
    
    // استدعاء API الذكاء الاصطناعي
    $apiKey = "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";
    $url = "https://api.openai.com/v1/completions";
    
    $data = [
        "model" => "gpt-4",
        "prompt" => "اكتب كودًا برمجيًا لمشروع: " . $projectDetails,
        "max_tokens" => 500
    ];
    
    $options = [
        'http' => [
            'header'  => "Content-type: application/json\r\nAuthorization: Bearer $apiKey\r\n",
            'method'  => 'POST',
            'content' => json_encode($data)
        ]
    ];
    
    $context  = stream_context_create($options);
    $result = file_get_contents($url, false, $context);
    
    if ($result === FALSE) {
        die("حدث خطأ أثناء الاتصال بواجهة الذكاء الاصطناعي.");
    }
    
    $response = json_decode($result, true);
    $generatedCode = $response['choices'][0]['text'] ?? '';
    
    if (empty($generatedCode)) {
        die("لم يتم توليد الكود بشكل صحيح.");
    }
    
    // إنشاء مجلد المشروع
    $projectFolder = "projects/project_" . time();
    mkdir($projectFolder, 0777, true);
    
    // حفظ الكود في ملف
    $filePath = "$projectFolder/main.php";
    file_put_contents($filePath, $generatedCode);
    
    // ضغط المجلد إلى ملف ZIP
    $zipFile = "$projectFolder.zip";
    $zip = new ZipArchive();
    if ($zip->open($zipFile, ZipArchive::CREATE) === TRUE) {
        $zip->addFile($filePath, "main.php");
        $zip->close();
    }
    
    echo "تم إنشاء المشروع بنجاح! يمكنك تحميله من <a href='$zipFile'>هنا</a>";
    exit;
}
?>
