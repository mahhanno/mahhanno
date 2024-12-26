<?php
$botToken = 'TOKEN-bot';// توکن ربات
$adminId = 'user-admin'; // آیدی عددی مدیر
$apiUrl = "https://api.telegram.org/bot$botToken/";

$content = file_get_contents("php://input");
$update = json_decode($content, true);

if (!$update || !isset($update['message'])) {
    exit;
}

$message = $update['message'];
$chatId = $message['chat']['id'];
$text = $message['text'] ?? '';

if (isset($message['reply_to_message']) && $chatId == $adminId) {
    // پاسخ مدیر به پیام کاربر
    $originalMessage = $message['reply_to_message']['text'];
    if (preg_match('/UserID:(\d+)/', $originalMessage, $matches)) {
        $userId = $matches[1];
        sendMessage($userId, $text);
        sendMessage($adminId, "✅ پیام شما به کاربر ارسال شد.");
    }
} elseif ($text == '/start') {
    // پیام خوش‌آمدگویی برای کاربر
    $welcomeText = "مرسی از mahan که منو طراحی کرد:) \n  هر سوالی داری بپرس تا به صورت ناشناس به دستش برسونم
     .";
    sendMessageWithButton($chatId, $welcomeText);
} else {
    // ارسال پیام کاربر به مدیر
    $forwardText = "📩 پیام جدید از کاربر:\n\n$text\n\nUserID:$chatId";
    sendMessage($adminId, $forwardText);

    // اطلاع‌رسانی به کاربر
    $confirmationText = "ارسال شد";
    sendMessageWithButton($chatId, $confirmationText);
}

function sendMessage($chatId, $text)
{
    global $apiUrl;
    $postData = [
        'chat_id' => $chatId,
        'text' => $text
    ];
    file_get_contents($apiUrl . "sendMessage?" . http_build_query($postData));
}

function sendMessageWithButton($chatId, $text)
{
    global $apiUrl;
    $postData = [
        'chat_id' => $chatId,
        'text' => $text,
        'reply_markup' => json_encode([
            'inline_keyboard' => [
                [
                    ['text' => 'mahan', 'url' => 'https://mahano.site'] // متن و لینک دکمه
                ]
            ]
        ])
    ];
    file_get_contents($apiUrl . "sendMessage?" . http_build_query($postData));
}
