# Show all appearance values

This is just a random post to showcase how to find all the properties of a (face) appearance. I created the rule to help someone on the Autodesk "[Inventor Programming Forum](https://forums.autodesk.com/t5/inventor-programming-forum/how-to-get-specific-texture-image-png-or-jpeg-file-path-used-on/td-p/13049899)". Someone else (or me) might find it useful in the future.

```vb.net
Sub main()

    Dim doc As PartDocument = ThisDoc.Document

    For Each asset As Asset In doc.Assets
        logger.Info("----------------------------------")
        logger.Info(asset.DisplayName.Trim())
        logger.Info("----------------------------------")


        LogAssetValues(asset.Cast(Of AssetValue))
    Next

End Sub

Public ReadOnly Property TexturePath As String
    Get
        Return ThisApplication.FileOptions.TexturePath
    End Get
End Property


Private Sub LogAssetValues(assetValues As IEnumerable(Of AssetValue), Optional preText As String = "")

    For Each assetValue As AssetValue In assetValues



        Select Case assetValue.ValueType
            Case AssetValueTypeEnum.kAssetValueTypeString
                PrintAssetValueDefault("String      : ", preText, assetValue)

            Case AssetValueTypeEnum.kAssetValueTypeBoolean
                PrintAssetValueDefault("Boolean     : ", preText, assetValue)

            Case AssetValueTypeEnum.kAssetValueTypeInteger
                PrintAssetValueDefault("Integer     : ", preText, assetValue)

            Case AssetValueTypeEnum.kAssetValueTypeFloat
                PrintAssetValueDefault("Float       : ", preText, assetValue)
                'logger.Info(preText + assetValue.DisplayName + ": " + assetValue.Value.ToString())

            Case AssetValueTypeEnum.kAssetValueTypeFilename
                Dim a As FilenameAssetValue = assetValue
                If (a.HasMultipleValues) Then
                    logger.Info("Filename    : " + preText + a.DisplayName + " (" + a.Name + ") has values: ")
                    For Each val As String In a.Values
                        logger.Info("            : " + preText + " - " + TexturePath + val)
                    Next
                Else
                    logger.Info("Filename    : " + preText + a.DisplayName + " (" + a.Name + ") has value: " + TexturePath + a.Value)
                End If


            Case AssetValueTypeEnum.kAssetValueTypeColor
                Dim a As ColorAssetValue = assetValue

                If (a.HasMultipleValues) Then
                    logger.Info("Color       : " + preText + a.DisplayName + " (" + a.Name + ") has values: -----------------------------------------------")
                    For Each color As Color In a.Values
                        Dim colorString = String.Format("Red:{0}, Green:{1}, Blue:{2}, Opacity:{3}",
                                                color.Red, color.Green, color.Blue, color.Opacity)

                        If (a.HasConnectedTexture) Then
                            logger.Info(preText + "  - " + a.DisplayName + ": " + colorString + "   (Has connected texture:)")
                            Dim textureAsset As AssetTexture = a.ConnectedTexture
                            LogAssetValues(textureAsset.Cast(Of AssetValue), "      " + preText)
                        Else
                            logger.Info("Color       : " + preText + a.DisplayName + " (" + a.Name + "): " + colorString + "   (NONE connected texture)")
                        End If
                    Next
                Else
                    Dim color = a.Value
                    Dim colorString = String.Format("Red:{0}, Green:{1}, Blue:{2}, Opacity:{3}",
                                                color.Red, color.Green, color.Blue, color.Opacity)

                    If (a.HasConnectedTexture) Then
                        logger.Info("Color       : " + preText + a.DisplayName + " (" + a.Name + "): " + colorString + "   Has connected texture:")
                        Dim textureAsset As AssetTexture = a.ConnectedTexture
                        LogAssetValues(textureAsset.Cast(Of AssetValue), "      " + preText)
                    Else
                        logger.Info("Color       : " + preText + a.DisplayName + " (" + a.Name + "): " + colorString + "   (NONE connected texture)")
                    End If
                End If

            Case AssetValueTypeEnum.kAssetValueTypeChoice
                Dim a As ChoiceAssetValue = assetValue

                Dim names As String() = {}
                Dim choises As String() = {}

                a.GetChoices(names, choises)
                Dim allChoises = String.Join("//", choises)

                logger.Info("Choice      : " + preText + a.DisplayName + " (" + a.Name + ") has value: " + a.Value + "(" + allChoises + ")")

            Case AssetValueTypeEnum.kAssetValueTextureType
                Dim a As TextureAssetValue = assetValue

                Dim texture As AssetTexture = a.Value

                logger.Info("Texture     : " + preText + a.DisplayName + " (" + a.Name + ")")
                LogAssetValues(texture.Cast(Of AssetValue), "      " + preText)

            Case AssetValueTypeEnum.kAssetValueTypeReference
                Dim a As ReferenceAssetValue = assetValue

                logger.Info("Reference   : " + preText + a.DisplayName + " (" + a.Name + "). The Value properies seem bugged")

                If (Not a.Value Is Nothing) Then
                    LogAssetValues(a.Value.Cast(Of AssetValue), "      " + preText)
                End If


                'Dim values As List(Of Asset) = New List(Of Asset)

                'If (a.HasMultipleValues) Then
                '    'Dim t As Object = a.Values
                '    'values = a.Values.AsEnumerable()
                '    'For Each asset As Asset In a.Values
                '    '    values.Add(asset)
                '    'Next

                'Else

                'End If

            Case Else

                logger.Info("----------> " + assetValue.DisplayName + ": " + CType(assetValue.Type, ObjectTypeEnum).ToString())
        End Select

    Next

End Sub

Private Sub PrintAssetValueDefault(type As String, preText As String, assetValue As AssetValue)
    Dim template = type + preText + "{0} ({1}): {2}"

    logger.Info(String.Format(template, assetValue.DisplayName, assetValue.Name, assetValue.Value.ToString()))
End Sub
```