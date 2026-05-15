package com.example.mcai;

import net.minecraft.client.gui.DrawContext;
import net.minecraft.client.gui.screen.Screen;
import net.minecraft.client.gui.widget.ButtonWidget;
import net.minecraft.client.gui.widget.TextFieldWidget;
import net.minecraft.text.Text;

import java.util.ArrayList;
import java.util.List;

public final class AiAssistantScreen extends Screen {
    private final Screen parent;
    private final List<String> chatLines = new ArrayList<>();

    private TextFieldWidget inputField;
    private int panelX;
    private int panelY;
    private int panelWidth;
    private int panelHeight;

    public AiAssistantScreen(Screen parent) {
        super(Text.literal("AI Assistant"));
        this.parent = parent;
        this.chatLines.add("AI: Ask about coords, health, items, time, help, or write any question.");
    }

    @Override
    protected void init() {
        this.panelWidth = Math.min(460, this.width - 40);
        this.panelHeight = Math.min(270, this.height - 40);
        this.panelX = (this.width - this.panelWidth) / 2;
        this.panelY = (this.height - this.panelHeight) / 2;

        int padding = 12;
        int buttonWidth = 58;
        int fieldHeight = 20;
        int bottomY = this.panelY + this.panelHeight - padding - fieldHeight;

        this.inputField = new TextFieldWidget(
                this.textRenderer,
                this.panelX + padding,
                bottomY,
                this.panelWidth - padding * 3 - buttonWidth,
                fieldHeight,
                Text.literal("Ask AI")
        );
        this.inputField.setMaxLength(256);
        this.inputField.setPlaceholder(Text.literal("Type message..."));
        this.inputField.setFocused(true);

        this.addDrawableChild(this.inputField);
        this.setInitialFocus(this.inputField);

        this.addDrawableChild(ButtonWidget.builder(Text.literal("Ask"), button -> submitPrompt())
                .dimensions(this.panelX + this.panelWidth - padding - buttonWidth, bottomY, buttonWidth, fieldHeight)
                .build());

        this.addDrawableChild(ButtonWidget.builder(Text.literal("Close"), button -> close())
                .dimensions(this.panelX + this.panelWidth - padding - 70, this.panelY + padding, 70, 20)
                .build());
    }

    private void submitPrompt() {
        if (this.inputField == null || this.client == null) {
            return;
        }

        String prompt = this.inputField.getText().trim();
        if (prompt.isEmpty()) {
            return;
        }

        addLine("You: " + prompt);
        addLine("AI: " + LocalAiBrain.answer(this.client, prompt));
        this.inputField.setText("");
    }

    private void addLine(String line) {
        for (String part : wrapLine(line, 66)) {
            this.chatLines.add(part);
        }

        while (this.chatLines.size() > 80) {
            this.chatLines.removeFirst();
        }
    }

    private static List<String> wrapLine(String line, int maxLength) {
        List<String> result = new ArrayList<>();
        String current = line;

        while (current.length() > maxLength) {
            int cut = current.lastIndexOf(' ', maxLength);
            if (cut < 20) {
                cut = maxLength;
            }
            result.add(current.substring(0, cut));
            current = current.substring(cut).trim();
        }

        if (!current.isEmpty()) {
            result.add(current);
        }

        return result;
    }

    @Override
    public void render(DrawContext context, int mouseX, int mouseY, float deltaTicks) {
        this.renderBackground(context, mouseX, mouseY, deltaTicks);

        context.fill(this.panelX, this.panelY, this.panelX + this.panelWidth, this.panelY + this.panelHeight, 0xCC111111);
        context.drawStrokedRectangle(this.panelX, this.panelY, this.panelWidth, this.panelHeight, 0xFF4FC3F7);
        context.drawTextWithShadow(this.textRenderer, "Minecraft AI Assistant", this.panelX + 12, this.panelY + 17, 0xFFFFFFFF);
        context.drawTextWithShadow(this.textRenderer, "Default key: .  Change it in Options > Controls > Key Binds", this.panelX + 12, this.panelY + 38, 0xFFBDBDBD);

        int chatX = this.panelX + 12;
        int chatY = this.panelY + 62;
        int maxVisibleLines = Math.max(1, (this.panelHeight - 102) / 10);
        int start = Math.max(0, this.chatLines.size() - maxVisibleLines);

        for (int i = start; i < this.chatLines.size(); i++) {
            String line = this.chatLines.get(i);
            int color = line.startsWith("You:") ? 0xFFFFF59D : 0xFFB3E5FC;
            context.drawTextWithShadow(this.textRenderer, line, chatX, chatY, color);
            chatY += 10;
        }

        super.render(context, mouseX, mouseY, deltaTicks);
    }

    @Override
    public boolean shouldPause() {
        return false;
    }

    @Override
    public void close() {
        if (this.client != null) {
            this.client.setScreen(this.parent);
        }
    }
}
