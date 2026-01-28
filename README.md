package com.DnDProject.org.DnD_Project;

import java.io.File;
import java.io.InputStream;
import java.io.FileWriter;
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import javafx.animation.FadeTransition;
import javafx.animation.KeyFrame;
import javafx.animation.Timeline;
import javafx.util.Duration;
import javafx.util.converter.DefaultStringConverter;
import javafx.application.Application;
import javafx.application.Platform;
import javafx.geometry.Pos;
import javafx.scene.Node;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.ContextMenu;
import javafx.scene.control.Label;
import javafx.scene.control.Labeled;
import javafx.scene.control.MenuItem;
import javafx.scene.control.ProgressBar;
import javafx.scene.control.ScrollPane;
import javafx.scene.control.TextField;
import javafx.scene.control.TreeItem;
import javafx.scene.control.TreeView;
import javafx.scene.control.cell.TextFieldTreeCell;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.input.ClipboardContent;
import javafx.scene.input.Dragboard;
import javafx.scene.input.TransferMode;
import javafx.scene.layout.FlowPane;
import javafx.scene.layout.HBox;
import javafx.scene.layout.Pane;
import javafx.scene.layout.StackPane;
import javafx.scene.layout.VBox;
import javafx.scene.paint.Color;
import javafx.scene.shape.Circle;
import javafx.scene.shape.Polygon;
import javafx.scene.shape.Rectangle;
import javafx.scene.shape.Shape;
import javafx.scene.text.Font;
import javafx.scene.text.FontWeight;
import javafx.scene.text.Text;
import javafx.stage.Stage;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;
import java.util.Scanner;
import java.util.concurrent.CompletableFuture;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;


/**
 * Developer: Nakealius-Ards
 * Date: 7 December 2025
 * Date Published: 28 January 2026
 * Application Title: DnD_Project
 * Purpose: This is an interactive DnD Model that allows the user to create and play a solo DnD Campaign.
 * Features include: Character, Attributes, Spells, Inventory creations with a character HUD, and a interactive text box
 * that generates a response powered by llama3.1 LLM.
 *
 * DnD_Project - Community Edition
 * 
 * This is an open-source interactive DnD model.
 * Contributors: Please keep file paths relative (e.g., assets/icons/...) 
 * so the project works cross-platform.
 * 
 * Contribution Guidelines:
 * - Document new features with clear comments.
 * - Avoid adding personal identifiers.
 * - Place new icons/images in the assets/icons directory.
 * - Extend functionality by adding new TreeItems for characters, abilities, spells, etc.
 */


public class DnD_Project extends Application {

	public static void main(String[] args) throws Exception {
		launch(args);
	}
	
	private File droppedImageFile;
	private ImageView imageView = new ImageView();
	private double offsetX;
    private double offsetY;
    private Stage arenaDisplayStage;
    private Pane arenaDisplayRoot; 
    private double currentHP = 80; 
    private double maxHP = 80; 
    private ProgressBar hpBar;
    private double damageHealInput;
    private Text hpPointsText;
	
	@Override
	public void start(Stage primaryStage) {
				
		//VBox for Characters, Abilities, Spells, and Inventory
		VBox casiVBox = new VBox();
		casiVBox.setAlignment(Pos.BASELINE_LEFT);
		casiVBox.setPrefHeight(1024);

		//TreeView Nodes and Controls
		TreeItem<String> casiMenu = new TreeItem<>("Menu");
		casiMenu.setExpanded(false); //expand by default
		
		//Add Child Nodes
		//Characters Nodes
		TreeItem<String> characters = new TreeItem<>("Characters");
		TreeItem<String> character01 = new TreeItem<>("Lucian Augustine");
		TreeItem<String> character02 = new TreeItem<>("Idris the Moonbound");
		//Add Abilities Node
		TreeItem<String> abilities = new TreeItem<>("Abilities");
		abilities.getChildren().add(new TreeItem<>("Rapier"));
		abilities.getChildren().add(new TreeItem<>("Bleeding Slash"));
		abilities.getChildren().add(new TreeItem<>("Dodge"));
		abilities.getChildren().add(new TreeItem<>("Counter"));
		abilities.getChildren().add(new TreeItem<>("Unarmed Strike"));
		abilities.getChildren().add(new TreeItem<>("Transformation"));
		abilities.getChildren().add(new TreeItem<>("Fury Slash"));
		
		TreeItem<String> buffs = new TreeItem<>("Buffs");
		buffs.getChildren().add(new TreeItem<>("+1 Armor"));
		buffs.getChildren().add(new TreeItem<>("+4 Damage"));
		buffs.getChildren().add(new TreeItem<>("+10 Speed/ft."));
		buffs.getChildren().add(new TreeItem<>("+18 Hit Points"));
		buffs.getChildren().add(new TreeItem<>("Damage Immunities: Bludgeoning, Piercing,\n and "
				+ "Slashing from weapons not \nmade of silver."));
		

		//Add Spells Node
		TreeItem<String> spells = new TreeItem<>("Spells");
		spells.getChildren().add(new TreeItem<>("Acid Splash"));
		spells.getChildren().add(new TreeItem<>("Guidance"));
		spells.getChildren().add(new TreeItem<>("Poison Spray"));
		spells.getChildren().add(new TreeItem<>("True Strike"));
		
		TreeItem<String> spells01 = new TreeItem<>("Spells");
		spells01.getChildren().add(new TreeItem<>("Bite"));
		spells01.getChildren().add(new TreeItem<>("Claw"));
		
		//Add Inventory Node
		TreeItem<String> inventory = new TreeItem<>("Inventory");
		inventory.getChildren().add(new TreeItem<>("Studded Leather Armor of Vulnerability"));
		inventory.getChildren().add(new TreeItem<>("Rapier"));
		inventory.getChildren().add(new TreeItem<>("Alchemist's Supplies"));
		inventory.getChildren().add(new TreeItem<>("Backpack with 8 slots"));
		
		TreeItem<String> lycanthropy = new TreeItem<>("Lycanthropy rules: ");
		lycanthropy.getChildren().add(new TreeItem<>("Transformation triggers: Full moon,\n emotional stress, or failed Wisdom saves."));
		lycanthropy.getChildren().add(new TreeItem<>("Control: May require Wisdom saves (>12) to\n resist involuntary transformation."));
		lycanthropy.getChildren().add(new TreeItem<>("Benefits: Natural weapons, damage \nimmunities, enhanced senses."));
		lycanthropy.getChildren().add(new TreeItem<>("Drawbacks: Loss of speech, vulnerability \nto silver, potential loss of control."));

		//Add the character's abilities, spells, and inventory
		characters.getChildren().add(character01);
		character01.getChildren().addAll(abilities, spells, inventory);
		
		characters.getChildren().add(character02);
		character02.getChildren().addAll(buffs, spells01, lycanthropy);
		//Add the Tree Branches to the Root
		casiMenu.getChildren().addAll(characters);
		
		//Create the TreeView with the Root
		TreeView<String> treeView = new TreeView<>(casiMenu);
		treeView.setPrefHeight(650);
		treeView.setPrefWidth(250);
		treeView.setEditable(true);
		treeView.setCellFactory(TextFieldTreeCell.forTreeView());
		treeView.setCellFactory(tv -> {
		    TextFieldTreeCell<String> cell = new TextFieldTreeCell<>(new DefaultStringConverter());

		    ContextMenu contextMenu = new ContextMenu();

		    MenuItem addItem = new MenuItem("Add Aspect");
		    addItem.setOnAction(e -> {
		        TreeItem<String> newChild = new TreeItem<>("New Item");
		        cell.getTreeItem().getChildren().add(newChild);
		    });

		    MenuItem deleteItem = new MenuItem("Delete");
		    deleteItem.setOnAction(e -> {
		        TreeItem<String> item = cell.getTreeItem();
		        if (item.getParent() != null) {
		            item.getParent().getChildren().remove(item);
		        }
		    });

		    contextMenu.getItems().addAll(addItem, deleteItem);

		    cell.setContextMenu(contextMenu);
		    return cell;
		});
		
		treeView.setOnEditCommit(event -> {
		    saveCASIState(event); // write to file or database
		});

		
		Button addCASIBtn = new Button("Add C.A.S.I");
		addCASIBtn.setLayoutX(85);
		addCASIBtn.setLayoutY(660);
		addCASIBtn.setOnAction(addCASIEvent -> {
			Stage addCASIStage = new Stage();
			addCASIItem addCASI = new addCASIItem();
			TreeItem<String> newCharacter = addCASI.start(addCASIStage);
			characters.getChildren().add(newCharacter);
		});
		
		//Character HUD Information HBox
		Pane characterPane = new Pane();
		characterPane.setPrefHeight(150);
		characterPane.setPrefWidth(1500);
		characterPane.setStyle("-fx-background-color: lightgrey;");
		
		// NOTE: All images/icons must be stored in assets/icons/ 
		// Replace file names with your own images if desired.
		//Shield Icon
		Image shieldIcon = new Image("file:assets/icons/shield-png.png");
		ImageView shieldImageView = new ImageView(shieldIcon);
		shieldImageView.setFitHeight(100);
		shieldImageView.setFitWidth(100);
		shieldImageView.setPreserveRatio(true);
		shieldImageView.setLayoutX(300);
		shieldImageView.setLayoutY(30);
		
		//Armor Class Text & Number
		Text armorClassText = new Text("Armor\n\n\n\n Class");
		armorClassText.setFont(Font.font("Verdana", FontWeight.BOLD, 20));
		armorClassText.setFill(Color.BLACK);
		armorClassText.setStroke(Color.GOLD);
		armorClassText.setLayoutX(310);
		armorClassText.setLayoutY(30);
		armorClassText.setWrappingWidth(100);
		
		int armorClassNumber = 20;
		String armorClassString = String.valueOf(armorClassNumber);
		Text armorClassNumberText = new Text(armorClassString);
		armorClassNumberText.setLayoutX(326);
		armorClassNumberText.setLayoutY(75);
		armorClassNumberText.setFont(Font.font("Verdana", FontWeight.BOLD, 28));
		armorClassNumberText.setFill(Color.BLACK);
		armorClassNumberText.setStroke(Color.GOLD);
		
		Button armorClassEdit = new Button();
		armorClassEdit.setLayoutX(326);
		
		
		// Character Photos
		Image lucianCharacterImage = new Image("file:assets/icons/Lucian Augustine Photo.PNG");
		Image idrisCharacterImage = new Image("file:assets/icons/Idris the Moonbound.png/");

		ImageView characterImageView = new ImageView(lucianCharacterImage);
		characterImageView.setFitHeight(100);
		characterImageView.setFitWidth(100);
		characterImageView.setLayoutX(550);
		characterImageView.setLayoutY(30);
		characterImageView.setPreserveRatio(false);

		// Store the names as Strings
		String lucianName = "Lucian Augustine";
		String idrisName = "Idris the Moonbound";

		// Create a single Text node that will be swapped
		Text characterText = new Text(lucianName);
		characterText.setFont(Font.font("Verdana", FontWeight.BOLD, 20));
		characterText.setFill(Color.BLUE);
		characterText.setStroke(Color.WHITE);
		characterText.setLayoutX(510);
		characterText.setLayoutY(18);

		Button transformBtn = new Button("Transform");
		transformBtn.setLayoutX(710);
		transformBtn.setLayoutY(90);

		transformBtn.setOnAction(e -> {
			// Fade out both image and text
		    FadeTransition fadeOutImage = new FadeTransition(Duration.millis(500), characterImageView);
		    fadeOutImage.setFromValue(1.0);
		    fadeOutImage.setToValue(0.0);

		    FadeTransition fadeOutText = new FadeTransition(Duration.millis(500), characterText);
		    fadeOutText.setFromValue(1.0);
		    fadeOutText.setToValue(0.0);

		    fadeOutImage.setOnFinished(ev -> {
		        // Swap image
		        if (characterImageView.getImage() == lucianCharacterImage) {
		            characterImageView.setImage(idrisCharacterImage);
		            characterText.setText(idrisName);
		            characterText.setFill(Color.RED);
		            characterText.setStroke(Color.BLACK);
		            characterText.setLayoutX(480);
		            maxHP = 98;
		            if (currentHP >= 80) {
		            	currentHP = 98;
		            } else {
		            	currentHP = currentHP + 18;
		            }
		            
		            updateHPBar();
		        } else {
		            characterImageView.setImage(lucianCharacterImage);
		            characterText.setText(lucianName);
		            characterText.setFill(Color.BLUE);
		            characterText.setStroke(Color.WHITE);
		            characterText.setLayoutX(510);
		            maxHP = 80;
		            if (currentHP >= 80) {
		            	currentHP = 80;		  
		            } else {
		            	currentHP = currentHP - 18;
		            }
		            
		            updateHPBar();
 
		        }

		        // Fade back in
		        FadeTransition fadeInImage = new FadeTransition(Duration.millis(500), characterImageView);
		        fadeInImage.setFromValue(0.0);
		        fadeInImage.setToValue(1.0);

		        FadeTransition fadeInText = new FadeTransition(Duration.millis(500), characterText);
		        fadeInText.setFromValue(0.0);
		        fadeInText.setToValue(1.0);

		        fadeInImage.play();
		        fadeInText.play();
		    });

		    fadeOutImage.play();
		    fadeOutText.play();
		});
		
		//HP Bar Buttons and Controls
		hpPointsText = new Text("HP: \n" + hpPoints());		
		hpPointsText.setLayoutX(660);
		hpPointsText.setLayoutY(100);
		hpPointsText.setFont(Font.font("Verdana", FontWeight.BOLD, 10));
		hpPointsText.setWrappingWidth(50);

		hpBar = new ProgressBar();
		hpBar.setLayoutX(250);
		hpBar.setLayoutY(0);
		hpBar.setPrefSize(775, 152);
		updateHPBar();
		
		TextField damageHealTextField = new TextField(null);
		damageHealTextField.setLayoutX(490);
		damageHealTextField.setLayoutY(60);
		damageHealTextField.setPrefSize(50, 50);
		damageHealTextField.setFont(Font.font("Verdana", FontWeight.BOLD, 18));
		
		Button hpHeal = new Button("Heal");
		hpHeal.setLayoutX(420);
		hpHeal.setLayoutY(58);
		hpHeal.setPrefWidth(60);
		
		hpHeal.setOnAction(healCharacter -> {
			try {
		        double value = Double.parseDouble(damageHealTextField.getText());
		        heal(value);
		        updateHPBar();
		    } catch (NumberFormatException e) {
		        System.out.println("Invalid input for healing!");
		    }
		});
		
		Button hpDamage = new Button("Damage");
		hpDamage.setLayoutX(420);
		hpDamage.setLayoutY(90);
		hpDamage.setOnAction(damageCharacter -> {
			try {
		        double value = Double.parseDouble(damageHealTextField.getText());
		        takeDamage(value);
		        updateHPBar();
		    } catch (NumberFormatException e) {
		        System.out.println("Invalid input for damage!");
		    }
		});
		
		
		characterPane.getChildren().addAll( hpBar, shieldImageView, armorClassText, armorClassNumberText, 
				hpHeal, hpDamage, damageHealTextField, hpPointsText, characterImageView, characterText, transformBtn);

		
		//ScrollPane for AI/UI interactions and interface
		VBox responseBox = new VBox(10);
        responseBox.setPrefWidth(740);
      //Load previous adventures
	    List<String> pastEvents = loadAdventureHistory();
	    for (String event : pastEvents) {
	        Label textLabel = new Label(event);
	        responseBox.getChildren().add(textLabel);
	    }

        ScrollPane aiScrollPane = new ScrollPane(responseBox);
        aiScrollPane.setMinWidth(755);
        aiScrollPane.setMinHeight(379);
        aiScrollPane.setMaxHeight(380);
        aiScrollPane.setLayoutX(260);
        aiScrollPane.setLayoutY(250);
        aiScrollPane.setFitToWidth(true);

        TextField aiPromptField = new TextField();
        aiPromptField.setPrefWidth(600);
        Button sendPromptBtn = new Button("Send");
        
        sendPromptBtn.setOnAction(sendPromptAction -> {
        	String prompt = aiPromptField.getText();
            if (!prompt.isEmpty()) {
                aiPromptField.clear();

                CompletableFuture.supplyAsync(() -> {
                    AiService aiService = new AiService();
                    return aiService.callOllamaChat(prompt);
                }).thenAccept(aiText -> {
                    Platform.runLater(() -> {
                        Label textLabel = new Label(aiText);
                        textLabel.setWrapText(true);
                        responseBox.getChildren().add(textLabel);
                        saveAdventure(prompt, aiText, "Lucian Augustine");
                    });
                });
            }
        });

        HBox aiInputBox = new HBox(10, aiPromptField, sendPromptBtn);
        aiInputBox.setLayoutX(310);
        aiInputBox.setLayoutY(650);
		
		//Map Button
		Button mapBtn = new Button("Map");
		mapBtn.setLayoutX(550);
		mapBtn.setLayoutY(200);
		mapBtn.setPrefWidth(80);
		mapBtn.setPrefHeight(10);
		mapBtn.setOnAction(displayMap -> {
			HBox mapDisplayHBox = new HBox();
			mapDisplayHBox.setSpacing(10);
			mapDisplayHBox.setPrefSize(550, 400);
			mapDisplayHBox.setLayoutX(25);
			mapDisplayHBox.setLayoutY(25);
			mapDisplayHBox.setStyle("-fx-border-color: gray; -fx-border-width: 2; -fx-alignment: center; -fx-background-color: #F0F0F0;");
			Label prompt = new Label("Drag and Drop an Image here...");
			mapDisplayHBox.getChildren().add(prompt);
			
			mapDisplayHBox.setOnDragOver(dragEvent -> {
				if (dragEvent.getGestureSource() != mapDisplayHBox && dragEvent.getDragboard().hasFiles()) {
					dragEvent.acceptTransferModes(TransferMode.COPY);
				}
				dragEvent.consume();
			});
			
			mapDisplayHBox.setOnDragDropped(dragEvent -> {
				if (dragEvent.getGestureSource() != mapDisplayHBox &&
						dragEvent.getDragboard().hasFiles()) {
					dragEvent.acceptTransferModes(javafx.scene.input.TransferMode.COPY);
				}
				dragEvent.consume();
			});
			
			mapDisplayHBox.setOnDragDropped(dragDroppedEvent -> {
				javafx.scene.input.Dragboard dragboard = dragDroppedEvent.getDragboard();
				boolean success = true;
				if (dragboard.hasFiles()) {
				    File file = dragboard.getFiles().get(0);
				    if (isImageFile(file)) {
				        droppedImageFile = file; //Store the file
				        Image image = new Image(file.toURI().toString());
				        imageView.setImage(image);
				        imageView.setFitWidth(550);
				        imageView.setFitHeight(400);
				        imageView.setPreserveRatio(false);
				        
				     // Center the image
				        double centerX = (mapDisplayHBox.getWidth() - imageView.getFitWidth()) / 2;
				        double centerY = (mapDisplayHBox.getHeight() - imageView.getFitHeight()) / 8;
				        imageView.setLayoutX(centerX);
				        imageView.setLayoutY(centerY);
				        
				        mapDisplayHBox.getChildren().setAll(imageView);
				    }
				}

				dragDroppedEvent.setDropCompleted(success);
				dragDroppedEvent.consume();
			});
			
			Button clearImgBtn = new Button("Clear Map");
			clearImgBtn.setLayoutX(240);
			clearImgBtn.setLayoutY(560);
			clearImgBtn.setPrefSize(120, 20);
			clearImgBtn.setOnAction(removeImg -> {
				imageView.setImage(null);
			});
			
			clearImgBtn.toFront();
			
			Pane mapDisplay = new Pane(mapDisplayHBox, clearImgBtn);
			
			
			Scene mapDisplayWindow = new Scene(mapDisplay, 600, 600);
			Stage mapDisplayStage = new Stage();
			mapDisplayStage.setScene(mapDisplayWindow);
			mapDisplayStage.setTitle("DnD Map Display");
			mapDisplayStage.show();
		});
		
		//Arena Button
		HBox arenaBtnHBox = new HBox();
		
		Button arenaBtn = new Button("Arena");
		arenaBtn.setLayoutX(640);
		arenaBtn.setLayoutY(200);
		arenaBtn.setPrefWidth(80);
		arenaBtn.setPrefHeight(10);
		arenaBtn.setOnAction(displayArena -> {
			
			if (arenaDisplayStage != null) {
		        arenaDisplayStage.show();
		        arenaDisplayStage.toFront();
		        return;
		    }
			
			// Otherwise create the stage for the first time
		    arenaDisplayStage = new Stage();

		    // Build Arena UI only once
		    arenaDisplayRoot = new Pane();
			
		    StackPane arenaDisplayHBox = new StackPane();
		    arenaDisplayHBox.setPrefSize(550, 400);
		    arenaDisplayHBox.setLayoutX(25);
		    arenaDisplayHBox.setLayoutY(25);
		    arenaDisplayHBox.setStyle("-fx-border-color: gray; -fx-border-width: 2; -fx-background-color: #F0F0F0;");

		    Label prompt = new Label("Drag and Drop an Image here...");
		    arenaDisplayHBox.getChildren().add(prompt);
			
			arenaDisplayHBox.setOnDragOver(dragEvent -> {
				if (dragEvent.getGestureSource() != arenaDisplayHBox && dragEvent.getDragboard().hasFiles()) {
					dragEvent.acceptTransferModes(TransferMode.COPY);
				}
				dragEvent.consume();
			});
			
			arenaDisplayHBox.setOnDragDropped(dragEvent -> {
				if (dragEvent.getGestureSource() != arenaDisplayHBox &&
						dragEvent.getDragboard().hasFiles()) {
					dragEvent.acceptTransferModes(javafx.scene.input.TransferMode.COPY);
				}
				dragEvent.consume();
			});
			
			arenaDisplayHBox.setOnDragDropped(dragDroppedEvent -> {
				javafx.scene.input.Dragboard dragboard = dragDroppedEvent.getDragboard();
				boolean success = true;
				if (dragboard.hasFiles()) {
				    File file = dragboard.getFiles().get(0);
				    if (isImageFile(file)) {
				        droppedImageFile = file; // ✅ Store the file
				        Image image = new Image(file.toURI().toString());
				        imageView.setImage(image);
				        imageView.setFitWidth(550);
				        imageView.setFitHeight(400);
				        imageView.setPreserveRatio(false);
				        
				     // Center the image
				        double centerX = (arenaDisplayHBox.getWidth() - imageView.getFitWidth()) / 2;
				        double centerY = (arenaDisplayHBox.getHeight() - imageView.getFitHeight()) / 8;
				        imageView.setLayoutX(centerX);
				        imageView.setLayoutY(centerY);
				        
				        arenaDisplayHBox.getChildren().setAll(imageView);
				    }
				}
				dragDroppedEvent.setDropCompleted(success);
				dragDroppedEvent.consume();
			});
			
			Button clearImgBtn = new Button("Clear Arena");
			clearImgBtn.setLayoutX(240);
			clearImgBtn.setLayoutY(560);
			clearImgBtn.setPrefSize(120, 20);
			clearImgBtn.setOnAction(removeImg -> {
				imageView.setImage(null);				
			});
			
			clearImgBtn.toFront();
			
			//Square Button with a small square icon
			Rectangle squareIcon = new Rectangle(35, 35);
			squareIcon.setFill(Color.TRANSPARENT);
			squareIcon.setStroke(Color.RED);
			squareIcon.setStrokeWidth(2);
			
			Button squareArenaBtn = new Button();
			squareArenaBtn.setLayoutX(50);
			squareArenaBtn.setLayoutY(450);
			squareArenaBtn.setPrefSize(70, 70);
			squareArenaBtn.setGraphic(squareIcon);
			squareArenaBtn.setOnAction(sqArena-> {
				// Clear previous shapes but keep the image if present
			    arenaDisplayHBox.getChildren().removeIf(node -> node instanceof Shape);
				Rectangle squareArena = new Rectangle(300, 300);
				squareArena.setLayoutX(100);
				squareArena.setLayoutY(100);
				
				//Transparent Fill
				squareArena.setFill(Color.TRANSPARENT);
				
				//Add a visible border
				squareArena.setStroke(Color.RED);
				squareArena.setStrokeWidth(3);
				
				arenaDisplayHBox.getChildren().add(squareArena);
			});
			
			Polygon octagonIcon = new Polygon();
			double radius = 20;
			double centerX = 20, centerY = 20;
			for (int i = 0; i < 8; i++) {
			    double angle = Math.toRadians(45 * i);
			    double x = centerX + radius * Math.cos(angle);
			    double y = centerY + radius * Math.sin(angle);
			    octagonIcon.getPoints().addAll(x, y);
			}
			
			octagonIcon.setFill(Color.TRANSPARENT);
			octagonIcon.setStroke(Color.RED);
			octagonIcon.setStrokeWidth(2);
			
			Button octagonArenaBtn = new Button();
			octagonArenaBtn.setLayoutX(140);
			octagonArenaBtn.setLayoutY(450);
			octagonArenaBtn.setPrefSize(70, 70);
			octagonArenaBtn.setGraphic(octagonIcon); // <-- put shape inside button
			octagonArenaBtn.setOnAction(octArena-> {
				// Clear previous shapes but keep the image if present
			    arenaDisplayHBox.getChildren().removeIf(node -> node instanceof Shape);
				
				double centerX1 = 150;
		        double centerY1 = 150;
		        double radius1 = 180;
		        
				Polygon octagonArena = new Polygon();
				for (int i = 0; i < 8; i++) {
		            double angle = Math.toRadians(45 * i); // 360/8 = 45 degrees
		            double x = centerX1 + radius1 * Math.cos(angle);
		            double y = centerY1 + radius1 * Math.sin(angle);
		            octagonArena.getPoints().addAll(x, y);
				
				octagonArena.setFill(Color.TRANSPARENT);
				
				octagonArena.setStroke(Color.RED);
				octagonArena.setStrokeWidth(3);
				}
				
				arenaDisplayHBox.getChildren().add(octagonArena);
				
			});
			
			Circle circleIcon = new Circle(20);
			circleIcon.setFill(Color.TRANSPARENT);
			circleIcon.setStroke(Color.RED);
			circleIcon.setStrokeWidth(2);
			
			Button circleArenaBtn = new Button();
			circleArenaBtn.setLayoutX(230);
			circleArenaBtn.setLayoutY(450);
			circleArenaBtn.setPrefSize(70, 70);
			circleArenaBtn.setGraphic(circleIcon); // <-- put shape inside button
			circleArenaBtn.setOnAction(cirArena-> {
				// Clear previous shapes but keep the image if present
			    arenaDisplayHBox.getChildren().removeIf(node -> node instanceof Shape);
				Circle circleArena = new Circle(200, 200, 180);
				circleArena.setLayoutY(100);
				
				//Transparent Fill
				circleArena.setFill(Color.TRANSPARENT);
				
				//Add a visible border
				circleArena.setStroke(Color.RED);
				circleArena.setStrokeWidth(3);
				
				arenaDisplayHBox.getChildren().add(circleArena);
				
			});
			
			arenaBtnHBox.setStyle("-fx-border-color: gray; -fx-border-width: 2; -fx-alignment: center; -fx-background-color: #F0F0F0;");
			arenaBtnHBox.setPrefSize(550, 80);
			arenaBtnHBox.setLayoutX(25);
			arenaBtnHBox.setLayoutY(445);
			
			//Werewolf Button
			File icon1File = new File("file:assets/icons/werewolf.png");
		    Image icon1Img = new Image(icon1File.toURI().toString());
		    ImageView icon1ImgView = new ImageView(icon1Img);
		    ImageView icon1ImgViewDuplicate = new ImageView(icon1Img);
		    icon1ImgViewDuplicate.setFitHeight(50);
		    icon1ImgViewDuplicate.setFitWidth(50);
		    icon1ImgView.setFitWidth(50);
		    icon1ImgView.setFitHeight(50);

		    Button werewolfBtn = new Button();
	    	Button werewolfBtnDuplicate = new Button();

		    werewolfBtn.setOnAction(duplicate -> {
		    	werewolfBtnDuplicate.setGraphic(icon1ImgViewDuplicate);
		    	werewolfBtnDuplicate.setLayoutX(50);
		    	werewolfBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(werewolfBtnDuplicate);
		    });
		    werewolfBtn.setPrefSize(70, 70);
		    werewolfBtn.setGraphic(icon1ImgView);
		    werewolfBtn.setLayoutX(310);
		    werewolfBtn.setLayoutY(450);
		    
		    werewolfBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - werewolfBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - werewolfBtnDuplicate.getLayoutY();
		    });
		    
		    werewolfBtnDuplicate.setOnMouseDragged(event -> {
		    	werewolfBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
		    	werewolfBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });

		    //Dragon Button
			File icon2File = new File("file:assets/icons/spiked-dragon-head.png/");
		    Image icon2Img = new Image(icon2File.toURI().toString());
		    ImageView icon2ImgView = new ImageView(icon2Img);
		    ImageView icon2ImgViewDuplicate = new ImageView(icon2Img);
		    icon2ImgViewDuplicate.setFitHeight(50);
		    icon2ImgViewDuplicate.setFitWidth(50);
		    icon2ImgView.setFitWidth(50);
		    icon2ImgView.setFitHeight(50);

		    Button dragonBtn = new Button();
	    	Button dragonBtnDuplicate = new Button();

		    dragonBtn.setOnAction(duplicate -> {
		    	dragonBtnDuplicate.setGraphic(icon2ImgViewDuplicate);
		    	dragonBtnDuplicate.setLayoutX(50);
		    	dragonBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(dragonBtnDuplicate);
		    });
		    dragonBtn.setPrefSize(70, 70);
		    dragonBtn.setGraphic(icon2ImgView);
		    dragonBtn.setLayoutX(310);
		    dragonBtn.setLayoutY(450);
		    
		    dragonBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - dragonBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - dragonBtnDuplicate.getLayoutY();
		    });
		    
		    dragonBtnDuplicate.setOnMouseDragged(event -> {
		    	dragonBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
		    	dragonBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Priest Button
			File icon3File = new File("file:assets/icons/pope-crown.png/");
		    Image icon3Img = new Image(icon3File.toURI().toString());
		    ImageView icon3ImgView = new ImageView(icon3Img);
		    ImageView icon3ImgViewDuplicate = new ImageView(icon3Img);
		    icon3ImgViewDuplicate.setFitHeight(50);
		    icon3ImgViewDuplicate.setFitWidth(50);
		    icon3ImgView.setFitWidth(50);
		    icon3ImgView.setFitHeight(50);

		    Button priestBtn = new Button();
	    	Button priestBtnDuplicate = new Button();

	    	priestBtn.setOnAction(duplicate -> {
	    		priestBtnDuplicate.setGraphic(icon3ImgViewDuplicate);
	    		priestBtnDuplicate.setLayoutX(50);
	    		priestBtnDuplicate.setLayoutY(50);
	    		arenaDisplayRoot.getChildren().add(priestBtnDuplicate);
		    });
	    	priestBtn.setPrefSize(70, 70);
	    	priestBtn.setGraphic(icon3ImgView);
	    	priestBtn.setLayoutX(310);
	    	priestBtn.setLayoutY(450);
		    
	    	priestBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - priestBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - priestBtnDuplicate.getLayoutY();
		    });
		    
	    	priestBtnDuplicate.setOnMouseDragged(event -> {
	    		priestBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		priestBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Minotaur Button
			File icon4File = new File("file:assets/icons/minotaur.png/");
		    Image icon4Img = new Image(icon4File.toURI().toString());
		    ImageView icon4ImgView = new ImageView(icon4Img);
		    ImageView icon4ImgViewDuplicate = new ImageView(icon4Img);
		    icon4ImgViewDuplicate.setFitHeight(50);
		    icon4ImgViewDuplicate.setFitWidth(50);
		    icon4ImgView.setFitWidth(50);
		    icon4ImgView.setFitHeight(50);

		    Button minotaurBtn = new Button();
	    	Button minotaurBtnDuplicate = new Button();

	    	minotaurBtn.setOnAction(duplicate -> {
	    		minotaurBtnDuplicate.setGraphic(icon4ImgViewDuplicate);
	    		minotaurBtnDuplicate.setLayoutX(50);
	    		minotaurBtnDuplicate.setLayoutY(50);
	    		arenaDisplayRoot.getChildren().add(minotaurBtnDuplicate);
		    });
	    	minotaurBtn.setPrefSize(70, 70);
	    	minotaurBtn.setGraphic(icon4ImgView);
	    	minotaurBtn.setLayoutX(310);
	    	minotaurBtn.setLayoutY(450);
		    
	    	minotaurBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - minotaurBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - minotaurBtnDuplicate.getLayoutY();
		    });
		    
	    	minotaurBtnDuplicate.setOnMouseDragged(event -> {
	    		minotaurBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		minotaurBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Goblin Button
			File icon5File = new File("file:assets/icons/goblin.png/");
		    Image icon5Img = new Image(icon5File.toURI().toString());
		    ImageView icon5ImgView = new ImageView(icon5Img);
		    ImageView icon5ImgViewDuplicate = new ImageView(icon5Img);
		    icon5ImgViewDuplicate.setFitHeight(50);
		    icon5ImgViewDuplicate.setFitWidth(50);
		    icon5ImgView.setFitWidth(50);
		    icon5ImgView.setFitHeight(50);

		    Button goblinBtn = new Button();
	    	Button goblinBtnDuplicate = new Button();

	    	goblinBtn.setOnAction(duplicate -> {
	    		goblinBtnDuplicate.setGraphic(icon5ImgViewDuplicate);
	    		goblinBtnDuplicate.setLayoutX(50);
	    		goblinBtnDuplicate.setLayoutY(50);
	    		arenaDisplayRoot.getChildren().add(goblinBtnDuplicate);
		    });
	    	goblinBtn.setPrefSize(70, 70);
	    	goblinBtn.setGraphic(icon5ImgView);
	    	goblinBtn.setLayoutX(310);
	    	goblinBtn.setLayoutY(450);
		    
	    	goblinBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - goblinBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - goblinBtnDuplicate.getLayoutY();
		    });
		    
	    	goblinBtnDuplicate.setOnMouseDragged(event -> {
	    		goblinBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		goblinBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Gluttonous Smile Button
			File icon6File = new File("file:assets/icons/gluttonous-smile.png/");
		    Image icon6Img = new Image(icon6File.toURI().toString());
		    ImageView icon6ImgView = new ImageView(icon6Img);
		    ImageView icon6ImgViewDuplicate = new ImageView(icon6Img);
		    icon6ImgViewDuplicate.setFitHeight(50);
		    icon6ImgViewDuplicate.setFitWidth(50);
		    icon6ImgView.setFitWidth(50);
		    icon6ImgView.setFitHeight(50);

		    Button gluttonBtn = new Button();
	    	Button gluttonBtnDuplicate = new Button();

	    	gluttonBtn.setOnAction(duplicate -> {
	    		gluttonBtnDuplicate.setGraphic(icon6ImgViewDuplicate);
	    		gluttonBtnDuplicate.setLayoutX(50);
	    		gluttonBtnDuplicate.setLayoutY(50);
	    		arenaDisplayRoot.getChildren().add(gluttonBtnDuplicate);
		    });
	    	gluttonBtn.setPrefSize(70, 70);
	    	gluttonBtn.setGraphic(icon6ImgView);
	    	gluttonBtn.setLayoutX(310);
	    	gluttonBtn.setLayoutY(450);
		    
	    	gluttonBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - gluttonBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - gluttonBtnDuplicate.getLayoutY();
		    });
		    
	    	gluttonBtnDuplicate.setOnMouseDragged(event -> {
	    		gluttonBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		gluttonBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Diablo Button
			File icon7File = new File("file:assets/icons/diablo-skull.png/");
		    Image icon7Img = new Image(icon7File.toURI().toString());
		    ImageView icon7ImgView = new ImageView(icon7Img);
		    ImageView icon7ImgViewDuplicate = new ImageView(icon7Img);
		    icon7ImgViewDuplicate.setFitHeight(50);
		    icon7ImgViewDuplicate.setFitWidth(50);
		    icon7ImgView.setFitWidth(50);
		    icon7ImgView.setFitHeight(50);

		    Button diabloBtn = new Button();
	    	Button diabloBtnDuplicate = new Button();

	    	diabloBtn.setOnAction(duplicate -> {
	    		diabloBtnDuplicate.setGraphic(icon7ImgViewDuplicate);
	    		diabloBtnDuplicate.setLayoutX(50);
	    		diabloBtnDuplicate.setLayoutY(50);
	    		arenaDisplayRoot.getChildren().add(diabloBtnDuplicate);
		    });
	    	diabloBtn.setPrefSize(70, 70);
	    	diabloBtn.setGraphic(icon7ImgView);
	    	diabloBtn.setLayoutX(310);
	    	diabloBtn.setLayoutY(450);
		    
	    	diabloBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - diabloBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - diabloBtnDuplicate.getLayoutY();
		    });
		    
	    	diabloBtnDuplicate.setOnMouseDragged(event -> {
	    		diabloBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		diabloBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Minion Button
			File icon8File = new File("file:assets/icons/bully-minion.png/");
		    Image icon8Img = new Image(icon8File.toURI().toString());
		    ImageView icon8ImgView = new ImageView(icon8Img);
		    ImageView icon8ImgViewDuplicate = new ImageView(icon8Img);
		    icon8ImgViewDuplicate.setFitHeight(50);
		    icon8ImgViewDuplicate.setFitWidth(50);
		    icon8ImgView.setFitWidth(50);
		    icon8ImgView.setFitHeight(50);

		    Button minionBtn = new Button();
	    	Button minionBtnDuplicate = new Button();

	    	minionBtn.setOnAction(duplicate -> {
	    		minionBtnDuplicate.setGraphic(icon8ImgViewDuplicate);
	    		minionBtnDuplicate.setLayoutX(50);
	    		minionBtnDuplicate.setLayoutY(50);
	    		arenaDisplayRoot.getChildren().add(minionBtnDuplicate);
		    });
	    	minionBtn.setPrefSize(70, 70);
	    	minionBtn.setGraphic(icon8ImgView);
	    	minionBtn.setLayoutX(310);
	    	minionBtn.setLayoutY(450);
		    
	    	minionBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - minionBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - minionBtnDuplicate.getLayoutY();
		    });
		    
	    	minionBtnDuplicate.setOnMouseDragged(event -> {
	    		minionBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		minionBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Barbarian Button
			File icon9File = new File("file:assets/icons/barbarian.png/");
		    Image icon9Img = new Image(icon9File.toURI().toString());
		    ImageView icon9ImgView = new ImageView(icon9Img);
		    ImageView icon9ImgViewDuplicate = new ImageView(icon9Img);
		    icon9ImgViewDuplicate.setFitHeight(50);
		    icon9ImgViewDuplicate.setFitWidth(50);
		    icon9ImgView.setFitWidth(50);
		    icon9ImgView.setFitHeight(50);

		    Button barbarianBtn = new Button();
	    	Button barbarianBtnDuplicate = new Button();

	    	barbarianBtn.setOnAction(duplicate -> {
	    		barbarianBtnDuplicate.setGraphic(icon9ImgViewDuplicate);
	    		barbarianBtnDuplicate.setLayoutX(50);
	    		barbarianBtnDuplicate.setLayoutY(50);
	    		arenaDisplayRoot.getChildren().add(barbarianBtnDuplicate);
		    });
	    	barbarianBtn.setPrefSize(70, 70);
	    	barbarianBtn.setGraphic(icon9ImgView);
	    	barbarianBtn.setLayoutX(310);
	    	barbarianBtn.setLayoutY(450);
		    
	    	barbarianBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - barbarianBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - barbarianBtnDuplicate.getLayoutY();
		    });
		    
	    	barbarianBtnDuplicate.setOnMouseDragged(event -> {
	    		barbarianBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		barbarianBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
		    //Gnome Button
			File icon10File = new File("file:assets/icons/bad-gnome.png/");
		    Image icon10Img = new Image(icon10File.toURI().toString());
		    ImageView icon10ImgView = new ImageView(icon10Img);
		    ImageView icon10ImgViewDuplicate = new ImageView(icon10Img);
		    icon10ImgViewDuplicate.setFitHeight(50);
		    icon10ImgViewDuplicate.setFitWidth(50);
		    icon10ImgView.setFitWidth(50);
		    icon10ImgView.setFitHeight(50);

		    Button gnomeBtn = new Button();
	    	Button gnomeBtnDuplicate = new Button();

	    	gnomeBtn.setOnAction(duplicate -> {
	    		gnomeBtnDuplicate.setGraphic(icon10ImgViewDuplicate);
	    		gnomeBtnDuplicate.setLayoutX(50);
	    		gnomeBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(gnomeBtnDuplicate);
		    });
	    	gnomeBtn.setPrefSize(70, 70);
	    	gnomeBtn.setGraphic(icon10ImgView);
	    	gnomeBtn.setLayoutX(310);
	    	gnomeBtn.setLayoutY(450);
		    
	    	gnomeBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - gnomeBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - gnomeBtnDuplicate.getLayoutY();
		    });
		    
	    	gnomeBtnDuplicate.setOnMouseDragged(event -> {
	    		gnomeBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		gnomeBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
	    	
	    	//Wizard Button
			File icon11File = new File("file:assets/icons/wizard-staff.png/");
		    Image icon11Img = new Image(icon11File.toURI().toString());
		    ImageView icon11ImgView = new ImageView(icon11Img);
		    ImageView icon11ImgViewDuplicate = new ImageView(icon11Img);
		    icon11ImgViewDuplicate.setFitHeight(50);
		    icon11ImgViewDuplicate.setFitWidth(50);
		    icon11ImgView.setFitWidth(50);
		    icon11ImgView.setFitHeight(50);

		    Button wizardBtn = new Button();
	    	Button wizardBtnDuplicate = new Button();

	    	wizardBtn.setOnAction(duplicate -> {
	    		wizardBtnDuplicate.setGraphic(icon11ImgViewDuplicate);
	    		wizardBtnDuplicate.setLayoutX(50);
	    		wizardBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(wizardBtnDuplicate);
		    });
	    	wizardBtn.setPrefSize(70, 70);
	    	wizardBtn.setGraphic(icon11ImgView);
	    	wizardBtn.setLayoutX(310);
	    	wizardBtn.setLayoutY(450);
		    
	    	wizardBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - wizardBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - wizardBtnDuplicate.getLayoutY();
		    });
		    
	    	wizardBtnDuplicate.setOnMouseDragged(event -> {
	    		wizardBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		wizardBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
	    	
	    	//Warlock Button
			File icon12File = new File("file:assets/icons/robe.png/");
		    Image icon12Img = new Image(icon12File.toURI().toString());
		    ImageView icon12ImgView = new ImageView(icon12Img);
		    ImageView icon12ImgViewDuplicate = new ImageView(icon12Img);
		    icon12ImgViewDuplicate.setFitHeight(50);
		    icon12ImgViewDuplicate.setFitWidth(50);
		    icon12ImgView.setFitWidth(50);
		    icon12ImgView.setFitHeight(50);

		    Button warlockBtn = new Button();
	    	Button warlockBtnDuplicate = new Button();

	    	warlockBtn.setOnAction(duplicate -> {
	    		warlockBtnDuplicate.setGraphic(icon12ImgViewDuplicate);
	    		warlockBtnDuplicate.setLayoutX(50);
	    		warlockBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(warlockBtnDuplicate);
		    });
	    	warlockBtn.setPrefSize(70, 70);
	    	warlockBtn.setGraphic(icon12ImgView);
	    	warlockBtn.setLayoutX(310);
	    	warlockBtn.setLayoutY(450);
		    
	    	warlockBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - warlockBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - warlockBtnDuplicate.getLayoutY();
		    });
		    
	    	warlockBtnDuplicate.setOnMouseDragged(event -> {
	    		warlockBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		warlockBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
	    	
	    	//Ranger Button
			File icon13File = new File("file:assets/icons/high-shot.png/");
		    Image icon13Img = new Image(icon13File.toURI().toString());
		    ImageView icon13ImgView = new ImageView(icon13Img);
		    ImageView icon13ImgViewDuplicate = new ImageView(icon13Img);
		    icon13ImgViewDuplicate.setFitHeight(50);
		    icon13ImgViewDuplicate.setFitWidth(50);
		    icon13ImgView.setFitWidth(50);
		    icon13ImgView.setFitHeight(50);

		    Button rangerBtn = new Button();
	    	Button rangerBtnDuplicate = new Button();

	    	rangerBtn.setOnAction(duplicate -> {
	    		rangerBtnDuplicate.setGraphic(icon13ImgViewDuplicate);
	    		rangerBtnDuplicate.setLayoutX(50);
	    		rangerBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(rangerBtnDuplicate);
		    });
	    	rangerBtn.setPrefSize(70, 70);
	    	rangerBtn.setGraphic(icon13ImgView);
	    	rangerBtn.setLayoutX(310);
	    	rangerBtn.setLayoutY(450);
		    
	    	rangerBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - rangerBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - rangerBtnDuplicate.getLayoutY();
		    });
		    
	    	rangerBtnDuplicate.setOnMouseDragged(event -> {
	    		rangerBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		rangerBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
	    	
	    	//Rogue Button
			File icon14File = new File("file:assets/icons/rogue.png/");
		    Image icon14Img = new Image(icon14File.toURI().toString());
		    ImageView icon14ImgView = new ImageView(icon14Img);
		    ImageView icon14ImgViewDuplicate = new ImageView(icon14Img);
		    icon14ImgViewDuplicate.setFitHeight(50);
		    icon14ImgViewDuplicate.setFitWidth(50);
		    icon14ImgView.setFitWidth(50);
		    icon14ImgView.setFitHeight(50);

		    Button rogueBtn = new Button();
	    	Button rogueBtnDuplicate = new Button();

	    	rogueBtn.setOnAction(duplicate -> {
	    		rogueBtnDuplicate.setGraphic(icon14ImgViewDuplicate);
	    		rogueBtnDuplicate.setLayoutX(50);
	    		rogueBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(rogueBtnDuplicate);
		    });
	    	rogueBtn.setPrefSize(70, 70);
	    	rogueBtn.setGraphic(icon14ImgView);
	    	rogueBtn.setLayoutX(310);
	    	rogueBtn.setLayoutY(450);
		    
	    	rogueBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - rogueBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - rogueBtnDuplicate.getLayoutY();
		    });
		    
	    	rogueBtnDuplicate.setOnMouseDragged(event -> {
	    		rogueBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		rogueBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    
	    	//Warrior Button
			File icon15File = new File("file:assets/icons/heavy-helm.png/");
		    Image icon15Img = new Image(icon15File.toURI().toString());
		    ImageView icon15ImgView = new ImageView(icon15Img);
		    ImageView icon15ImgViewDuplicate = new ImageView(icon15Img);
		    icon15ImgViewDuplicate.setFitHeight(50);
		    icon15ImgViewDuplicate.setFitWidth(50);
		    icon15ImgView.setFitWidth(50);
		    icon15ImgView.setFitHeight(50);

		    Button warriorBtn = new Button();
	    	Button warriorBtnDuplicate = new Button();

	    	warriorBtn.setOnAction(duplicate -> {
	    		warriorBtnDuplicate.setGraphic(icon15ImgViewDuplicate);
	    		warriorBtnDuplicate.setLayoutX(50);
	    		warriorBtnDuplicate.setLayoutY(50);
		    	arenaDisplayRoot.getChildren().add(warriorBtnDuplicate);
		    });
	    	warriorBtn.setPrefSize(70, 70);
	    	warriorBtn.setGraphic(icon15ImgView);
	    	warriorBtn.setLayoutX(310);
	    	warriorBtn.setLayoutY(450);
		    
	    	warriorBtnDuplicate.setOnMousePressed(event -> {
		    	offsetX = event.getSceneX() - warriorBtnDuplicate.getLayoutX();
	            offsetY = event.getSceneY() - warriorBtnDuplicate.getLayoutY();
		    });
		    
	    	warriorBtnDuplicate.setOnMouseDragged(event -> {
	    		warriorBtnDuplicate.setLayoutX(event.getSceneX() - offsetX);
	    		warriorBtnDuplicate.setLayoutY(event.getSceneY() - offsetY);
		    });
		    //Add the buttons to the HBox Container
		    arenaBtnHBox.getChildren().addAll(squareArenaBtn, octagonArenaBtn, circleArenaBtn, werewolfBtn, dragonBtn, priestBtn, minotaurBtn,
		    		goblinBtn, gluttonBtn, diabloBtn, minionBtn, barbarianBtn, gnomeBtn, warlockBtn, wizardBtn, rangerBtn, rogueBtn, warriorBtn);

		    //ScrollPane for the Buttons
		    ScrollPane arenaBtnScroll = new ScrollPane(arenaBtnHBox);
		    arenaBtnScroll.setLayoutX(25);
		    arenaBtnScroll.setLayoutY(450);
		    
			arenaDisplayRoot.getChildren().addAll(arenaDisplayHBox, arenaBtnScroll, clearImgBtn);

			Scene arenaScene = new Scene(arenaDisplayRoot, 600, 600);
		    arenaDisplayStage.setScene(arenaScene);
		    arenaDisplayStage.setTitle("DnD Arena Display");
			arenaDisplayStage.show();
		});
		
		Pane mainWindow = new Pane(characterPane, casiVBox, aiScrollPane, aiInputBox, mapBtn, arenaBtn, treeView, addCASIBtn);
		Scene scene = new Scene(mainWindow, 1024, 700);
        primaryStage.setTitle("DnD Project");
        primaryStage.setScene(scene);
        primaryStage.show();
	}
	
	private List<String> loadAdventureHistory() {
		try {
	        List<String> allLines = Files.readAllLines(Paths.get("adventures.txt"));
	        int total = allLines.size();
	        int startIndex = (int) Math.floor(total * 0.75); // last 25%
	        return allLines.subList(startIndex, total);
	    } catch (Exception e) {
	        e.printStackTrace();
	        return List.of(); // empty list if error
	    }
	}

	private boolean isImageFile(File file) {
		String name = file.getName().toLowerCase();
		return name.endsWith(".png") || name.endsWith(".jpg") || name.endsWith("jpeg") || name.endsWith(".gif");
	}
	
	public void saveAdventure(String prompt, String response, String characterName) {
	    try (FileWriter writer = new FileWriter("adventures.txt", true)) {
	        // Format timestamp
	        String timestamp = LocalDateTime.now()
	                .format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

	        // Write structured entry
	        writer.write("=== Adventure Log ===\n");
	        writer.write("Time: " + timestamp + "\n");
	        writer.write("Character: " + characterName + "\n");
	        writer.write("Prompt: " + prompt + "\n");
	        writer.write("Response:\n" + response + "\n\n");
	    } catch (IOException e) {
	        e.printStackTrace();
	    }
	}
	
	// AI SERVICE NOTE:
	// This project uses AiService.callOllamaChat(prompt).
	// Contributors can replace AiService with other LLM backends if desired.
	class AiService {
	    private final HttpClient client = HttpClient.newHttpClient();
	    private final ObjectMapper objectMapper = new ObjectMapper();

	    public String callOllamaChat(String prompt) {
	        try {
	        	String requestBody = """
	        			{
	        			  "model": "llama3.1:latest",
	        			  "prompt": "%s"
	        			}
	        			""".formatted(prompt);

	            HttpRequest request = HttpRequest.newBuilder()
	                    .uri(URI.create("http://localhost:11434/api/generate"))
	                    .header("Content-Type", "application/json")
	                    .POST(HttpRequest.BodyPublishers.ofString(requestBody))
	                    .build();

	            HttpResponse<InputStream> response =
	            	    client.send(request, HttpResponse.BodyHandlers.ofInputStream());

	            	StringBuilder sb = new StringBuilder();
	            	try (Scanner scanner = new Scanner(response.body(), StandardCharsets.UTF_8)) {
	            	    while (scanner.hasNextLine()) {
	            	        String line = scanner.nextLine();
	            	        JsonNode node = objectMapper.readTree(line);

	            	        if (node.has("error")) {
	            	            return "Error: " + node.get("error").asText();
	            	        }
	            	        if (node.has("response")) {
	            	            sb.append(node.get("response").asText());
	            	        }
	            	        if (node.has("done") && node.get("done").asBoolean()) {
	            	            break; // stop when Ollama signals completion
	            	        }
	            	    }
	            	}
	            	return sb.toString();

	        } catch (Exception e) {
	            e.printStackTrace();
	            return "Error calling Ollama: " + e.getMessage();
	        }
	    }
	}

	// COMMUNITY TIP: Add new characters by creating TreeItem<String>("CharacterName")
	// and attaching abilities, spells, and inventory nodes.
	public class addCASIItem {

	    public TreeItem<String> start(Stage CASIInfoStage) {

	        Text characterNameText = new Text("Name:\t");
	        characterNameText.setLayoutX(100);
	        characterNameText.setLayoutY(29);
	        TextField characterNameTextField = new TextField("Enter your character name...");
	        characterNameTextField.setLayoutX(140);
	        characterNameTextField.setLayoutY(10);

	        FlowPane attributeFlow = new FlowPane();
	        attributeFlow.setLayoutY(80);

	        FlowPane spellFlow = new FlowPane();
	        spellFlow.setLayoutY(250);

	        FlowPane inventoryFlow = new FlowPane();
	        inventoryFlow.setLayoutY(420);
	        
	        TextField attributeTextField = new TextField();
	        attributeTextField.setLayoutX(110);
	        attributeTextField.setLayoutY(50);

	        Button addAttributeBtn = new Button("Add Attribute");
	        addAttributeBtn.setLayoutX(10);
	        addAttributeBtn.setLayoutY(50);
	        addAttributeBtn.setOnAction(e -> {
	            Text t = new Text(attributeTextField.getText()+ "; ");
	            attributeFlow.getChildren().add(t);
	        });

	        TextField spellsTextField = new TextField();
	        spellsTextField.setLayoutX(90);
	        spellsTextField.setLayoutY(220);
	        
	        Button addSpellBtn = new Button("Add Spell");
	        addSpellBtn.setLayoutX(10);
	        addSpellBtn.setLayoutY(220);
	        addSpellBtn.setOnAction(e -> {
	            Text t = new Text(spellsTextField.getText()+ "; ");
	            spellFlow.getChildren().add(t);
	        });

	        TextField inventoryTextField = new TextField();
	        inventoryTextField.setLayoutX(90);
	        inventoryTextField.setLayoutY(390);
	        
	        Button addInventoryBtn = new Button("Add Item");
	        addInventoryBtn.setLayoutX(10);
	        addInventoryBtn.setLayoutY(390);
	        addInventoryBtn.setOnAction(e -> {
	            Text t = new Text(inventoryTextField.getText()+ "; ");
	            inventoryFlow.getChildren().add(t);
	        });

	        // Returned container
	        TreeItem<String> characterRoot = new TreeItem<>();  

	        TreeItem<String> abilities = new TreeItem<>("Abilities");
	        TreeItem<String> spells = new TreeItem<>("Spells");
	        TreeItem<String> items = new TreeItem<>("Inventory");

	        Button addAllBtn = new Button("Add All");
	        addAllBtn.setLayoutX(180);
	        addAllBtn.setLayoutY(550);
	        addAllBtn.setOnAction(e -> {

	            characterRoot.setValue(characterNameTextField.getText());

	            for (Node n : attributeFlow.getChildren()) {
	                Text t = (Text) n;
	                abilities.getChildren().add(new TreeItem<>(t.getText()));
	            }
	            for (Node n : spellFlow.getChildren()) {
	                Text t = (Text) n;
	                spells.getChildren().add(new TreeItem<>(t.getText()));
	            }
	            for (Node n : inventoryFlow.getChildren()) {
	                Text t = (Text) n;
	                items.getChildren().add(new TreeItem<>(t.getText()));
	            }

	            characterRoot.getChildren().addAll(abilities, spells, items);

	            CASIInfoStage.close();
	        });
	        

	        Pane root = new Pane(characterNameText, characterNameTextField,
	                addAttributeBtn, attributeTextField, attributeFlow,
	                addSpellBtn, spellsTextField, spellFlow,
	                addInventoryBtn, inventoryTextField, inventoryFlow,
	                addAllBtn);

	        CASIInfoStage.setScene(new Scene(root, 400, 600));
	        CASIInfoStage.show();

	        return characterRoot;
	    }
	}
	
	private List<String> saveCASIState(TreeView.EditEvent<String> event) {
	    TreeItem<String> item = event.getTreeItem();
	    item.setValue(event.getNewValue());

	    // Append the updated value to the file
	    try (FileWriter writer = new FileWriter("casiUpdates.txt", true)) {
	        writer.write(item.getValue() + System.lineSeparator());
	    } catch (IOException e) {
	        e.printStackTrace();
	    }

	    // Read back the file contents
	    try {
	        List<String> allLines = Files.readAllLines(Paths.get("casiUpdates.txt"));
	        return allLines;
	    } catch (IOException e) {
	        e.printStackTrace();
	        return List.of(); // empty list if error
	    }
	}
	
	private void updateHPBar() {
	    double progress = currentHP / maxHP;
	    if (progress < 0.3) {
	        hpBar.setStyle("-fx-accent: red;");
	    } else if (progress < 0.7) {
	        hpBar.setStyle("-fx-accent: orange;");
	    } else if (progress > 0.7){
	        hpBar.setStyle("-fx-accent: lightgreen;");
	    }
	    animateHPBar(progress);
	    
		hpPointsText.setText("HP:\n" + hpPoints());
	}

	private String hpPoints() { 
		return String.format("%.0f / %.0f", currentHP, maxHP); 
	}
	
	public void heal(double amount) {
	    currentHP = Math.min(maxHP, currentHP + amount); 
	    updateHPBar();
	}

	public void takeDamage(double amount) {
	    currentHP = Math.max(0, currentHP - amount);
	    updateHPBar();
	}
	
	private void animateHPBar(double targetProgress) {
	    Timeline timeline = new Timeline(
	        new KeyFrame(Duration.millis(300),
	            new javafx.animation.KeyValue(hpBar.progressProperty(), targetProgress))
	    );
	    timeline.play();
	}  
}

//TODO (Community Ideas):
//- Add multiplayer support
//- Expand spell/ability sets
//- Create save/load functionality for campaigns
